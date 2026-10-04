# Laboratorio 1 — Redes virtuales con Linux

## 1. Datos del entorno

| Campo | Valor |
|---|---|
| Estudiante | Josue Llanque |
| Fecha | 2026-10-02 (reconocimiento inicial: 2026-09-25) |
| Entorno | WSL2 sobre Windows |
| Distribución | Ubuntu 26.04.1 LTS (Resolute Raccoon) |
| Kernel | 6.18.33.2-microsoft-standard-WSL2 (x86_64) |
| Usuario / host | `jllanque` / `MSI` |

Evidencia: `evidencias/00-entorno.txt`.

## 2. Objetivo

Construir una red virtual aislada con *network namespaces* y un par `veth`,
asignar direccionamiento IPv4, verificar la conectividad, observar
interfaces, rutas, vecinos y tráfico, provocar una falla controlada y
diagnosticarla.

## 3. Topología

```text
             Linux

       hostA                           hostB
┌───────────────────┐          ┌───────────────────┐
│ network namespace │          │ network namespace │
│  10.10.1.1/30     │          │  10.10.1.2/30     │
│       vethA       ├══════════┤       vethB       │
└───────────────────┘   veth   └───────────────────┘
```

Una sola computadora representa dos nodos independientes: cada namespace
tiene su propia pila de red y el par `veth` actúa como un cable Ethernet
virtual entre ellos.

## 4. Direccionamiento

| Elemento | Valor |
|---|---|
| Namespace 1 | `hostA` |
| Namespace 2 | `hostB` |
| Interfaz de hostA | `vethA` |
| Interfaz de hostB | `vethB` |
| Dirección hostA | `10.10.1.1/30` |
| Dirección hostB | `10.10.1.2/30` |

Objetivo: obtener conectividad IPv4 entre `hostA` y `hostB`.

Subred `10.10.1.0/30` (2 bits de host → 4 direcciones):

| Dirección | Tipo |
|---|---|
| 10.10.1.0 | Red |
| 10.10.1.1 | Host utilizable (hostA) |
| 10.10.1.2 | Host utilizable (hostB) |
| 10.10.1.3 | Broadcast |

Hay **2 direcciones utilizables** (2² − 2). Ambas direcciones, al aplicar la
máscara 255.255.255.252, dan la misma red `10.10.1.0`, y están conectadas al
mismo enlace (el veth): por eso son vecinos directos.

## 5. Construcción

### 5.1 Reconocimiento de la red (Actividad 1)

Sesión del 25/09. Interfaces observadas:

- `lo`: loopback, UP, `127.0.0.1/8` y `10.255.255.254/32`.
- `eth0`: interfaz de red WSL2, UP, `172.26.58.217/20`, MAC `00:15:5d:f2:82:e8`, MTU 1472.

Ruta por defecto:

- `default via 172.26.48.1 dev eth0`.

Vecino conocido:

- `172.26.48.1` (gateway), MAC `00:15:5d:37:4f:dd`, estado REACHABLE.

Evidencia: `evidencias/01-reconocimiento.txt`.

**Repetición el 02/10.** La IP y el prefijo (`172.26.58.217/20`) y el gateway
(`172.26.48.1`) se mantuvieron, pero WSL2 cambió otros valores: `eth0` pasó a
MAC `00:15:5d:de:88:5d` y MTU 1500, y el gateway a MAC `00:15:5d:4e:d4:fb` con
estado STALE. En esa sesión `ping -c 4 8.8.8.8` y `ping -c 4 google.com`
respondieron con 0 % de pérdida (`google.com` resolvió a `142.250.0.139`), y
`traceroute 8.8.8.8` llegó en 12 saltos. El primer salto fue
`MSI.mshome.net (172.26.48.1)`, es decir, el propio Windows: el gateway de
WSL2 es el host.

### 5.2 Pregunta de diagnóstico

Si `ping 8.8.8.8` funciona pero `ping google.com` no, el componente
a investigar primero es **DNS**. La resolución de nombres (`google.com`
→ IP) depende del resolvedor configurado en `/etc/resolv.conf`. La
conectividad IP básica ya está probada con 8.8.8.8.

### 5.3 Namespaces

```bash
sudo ip netns add hostA
sudo ip netns add hostB
ip netns list
```

`ip netns list` mostró `hostB` y `hostA`. Un *network namespace* aísla toda la
pila de red de los procesos que lo habitan: interfaces, direcciones, rutas,
tabla de vecinos, sockets y reglas de firewall. No es una máquina virtual:
comparte el kernel con el sistema. Un namespace nuevo solo tiene `lo` en
estado DOWN.

### 5.4 Par veth

```bash
sudo ip link add vethA type veth peer name vethB
ip link
```

Tras crearlo, `vethA@vethB` y `vethB@vethA` aparecieron en el espacio de red
principal, ambos en `state DOWN`. Aparecen ahí porque una interfaz se crea en
el namespace desde el que se ejecuta el comando, y el par se crea completo, con
los dos extremos a la vez.

### 5.5 Mover cada extremo

```bash
sudo ip link set vethA netns hostA
sudo ip link set vethB netns hostB
```

Después de mover, el `ip link` principal volvió a mostrar solo `lo` y `eth0`:
una interfaz pertenece a un único namespace a la vez. `vethA` existe ahora en
`hostA` y `vethB` en `hostB`; el campo `link-netns` de cada una apunta al otro
namespace, lo que muestra que el par sigue conectado.

Observación: las MAC de los veth cambiaron al moverlos (por ejemplo, `vethA`
pasó de `ae:b5:26:2d:c3:b0` a `d6:5b:58:3e:fd:cf`). No determiné la causa; las
MAC que quedaron vigentes son `d6:5b:58:3e:fd:cf` (vethA) y
`56:cc:ee:03:87:1a` (vethB), y son las que aparecen después en ARP.

### 5.6 Direccionamiento y puesta en servicio

```bash
sudo ip netns exec hostA ip addr add 10.10.1.1/30 dev vethA
sudo ip netns exec hostB ip addr add 10.10.1.2/30 dev vethB
sudo ip netns exec hostA ip link set lo up
sudo ip netns exec hostA ip link set vethA up
sudo ip netns exec hostB ip link set lo up
sudo ip netns exec hostB ip link set vethB up
```

Con las direcciones asignadas pero las interfaces aún en DOWN, `ip route` de
hostA salió vacío. Al levantar las interfaces pasaron a `UP, LOWER_UP`.

**Existente, direccionada y `UP`.** Una interfaz *existente* es una que el
kernel conoce (aparece en `ip link`). *Direccionada* significa que tiene una IP
asignada; eso es configuración y no la enciende: `vethA` tenía
`10.10.1.1/30` y seguía en `state DOWN`. `UP` la habilita administrativamente, y
`LOWER_UP` indica que además hay carrier (en un veth, que el otro extremo
también esté arriba).

Evidencia: `evidencias/02-construccion.txt`.

### 5.7 Procedimiento reproducible

Resumen en orden para que otra persona pueda repetir la construcción (requiere
`iproute2` y permisos de administrador):

```bash
# 1. Namespaces (dos nodos aislados)
sudo ip netns add hostA
sudo ip netns add hostB

# 2. Cable virtual y traslado de cada extremo
sudo ip link add vethA type veth peer name vethB
sudo ip link set vethA netns hostA
sudo ip link set vethB netns hostB

# 3. Direccionamiento
sudo ip netns exec hostA ip addr add 10.10.1.1/30 dev vethA
sudo ip netns exec hostB ip addr add 10.10.1.2/30 dev vethB

# 4. Puesta en servicio
sudo ip netns exec hostA ip link set lo up
sudo ip netns exec hostA ip link set vethA up
sudo ip netns exec hostB ip link set lo up
sudo ip netns exec hostB ip link set vethB up

# 5. Verificación
sudo ip netns exec hostA ip route
sudo ip netns exec hostA ping -c 4 10.10.1.2
sudo ip netns exec hostB ping -c 4 10.10.1.1
sudo ip netns exec hostA ip neigh
```

Los namespaces creados con `ip netns add` no sobreviven a un reinicio de WSL;
tras reiniciar hay que repetir el procedimiento.

## 6. Verificación

### Interfaces

`vethA` en hostA y `vethB` en hostB, ambas `UP, LOWER_UP`, cada una junto a su
`lo`. Ver `evidencias/02-construccion.txt`.

### Direcciones

`10.10.1.1/30` en `vethA` y `10.10.1.2/30` en `vethB`.

### Rutas (Actividad 3)

```text
hostA: 10.10.1.0/30 dev vethA proto kernel scope link src 10.10.1.1
hostB: 10.10.1.0/30 dev vethB proto kernel scope link src 10.10.1.2
```

1. En hostA existe la ruta hacia `10.10.1.0/30` por `vethA`.
2. En hostB existe la ruta hacia `10.10.1.0/30` por `vethB`.
3. Aparecieron solas: es una *ruta conectada* que el kernel crea
   (`proto kernel`) al asignar IP y prefijo a una interfaz que está UP. Con la
   interfaz DOWN no existía.
4. Para llegar al otro nodo hostA usa `vethA` y hostB usa `vethB`.
5. **No hace falta gateway.** Un gateway sirve para alcanzar redes remotas; aquí
   el destino está en la misma subred y el mismo enlace (`scope link`), no hay
   `via` en la ruta y no existe ningún router intermedio.

### Vecinos (Actividad 4)

Antes del ping, la tabla de hostA estaba vacía. Después:

| Namespace | IP vecino | MAC | Estado |
|---|---|---|---|
| hostA | 10.10.1.2 | 56:cc:ee:03:87:1a | REACHABLE |
| hostB | 10.10.1.1 | d6:5b:58:3e:fd:cf | REACHABLE |

Las MAC coinciden con las de `vethB` y `vethA` en `ip link`. IPv4 necesita una
dirección de capa 2 porque Ethernet entrega tramas por MAC: el emisor conoce la
IP del destino, pero para construir la trama necesita su MAC, y ARP se la
proporciona.

### Conectividad

```text
hostA → 10.10.1.2: 4 transmitted, 4 received, 0% packet loss, rtt avg 0.071 ms
hostB → 10.10.1.1: 4 transmitted, 4 received, 0% packet loss, rtt avg 0.074 ms
```

Un `ttl=64` sin decrementar confirma que no hubo router intermedio. Las
latencias de décimas de milisegundo se explican porque el tráfico no sale a
ninguna red física.

Evidencia: `evidencias/03-rutas-vecinos-ping.txt`.

## 7. Captura y análisis de tráfico (Actividad 5)

Antes de capturar se vació la caché de vecinos (`ip neigh flush all`) en ambos
namespaces. Captura en `vethA` de hostA mientras hostB hacía ping:

```bash
sudo ip netns exec hostA tcpdump -n -e -i vethA
sudo ip netns exec hostB ping -c 4 10.10.1.1
```

Secuencia observada (15:39:46):

1. **ARP Request** en broadcast (`ff:ff:ff:ff:ff:ff`): `who-has 10.10.1.1 tell 10.10.1.2`.
2. **ARP Reply** en unicast: `10.10.1.1 is-at d6:5b:58:3e:fd:cf`.
3. **ICMP Echo Request** `10.10.1.2 > 10.10.1.1`, `seq 1`, tres microsegundos
   después del ARP.
4. **ICMP Echo Reply** `10.10.1.1 > 10.10.1.2`, `seq 1`.
5. `seq 2` a `seq 4` se repiten sin ARP: la MAC ya estaba en caché.

Primero ARP (resolución de la MAC) y después ICMP. Unos 2 segundos después del
último Echo Reply aparece una pareja ARP unicast adicional (`who-has 10.10.1.2
tell 10.10.1.1`): interpreto que el kernel revalida la entrada de vecino; es una
interpretación mía, no la verifiqué.

También aparecieron `ICMP6 router solicitation` hacia `ff02::2`, mensajes MLD y
un `neighbor solicitation`: es tráfico IPv6 (NDP) que el kernel genera al
levantar las interfaces, sin relación con el ping. No recibieron respuesta
porque en la topología no hay routers.

Evidencia: `evidencias/04-captura-tcpdump.txt`.

## 8. Falla y diagnóstico (Actividad 6)

### 8.1 Predicción previa

> ESCRIBE AQUÍ tu predicción, tal como la pensaste ANTES de ejecutar
> `ip link set vethB down`. Si no la escribiste en su momento, repite la falla
> (ver abajo) y regístrala antes de ejecutarla.

### 8.2 Síntoma

Tras `sudo ip netns exec hostB ip link set vethB down`, el ping de hostA a
`10.10.1.2` dio `From 10.10.1.1 icmp_seq=1 Destination Host Unreachable` y
`4 packets transmitted, 0 received, +1 errors, 100% packet loss`.

### 8.3 Diagnóstico

| Comando | Hallazgo |
|---|---|
| hostA `ip link` | `vethA`: `<NO-CARRIER,BROADCAST,MULTICAST,UP>`, `state DOWN` |
| hostA `ip addr` | `10.10.1.1/30` sigue asignada |
| hostA `ip route` | `10.10.1.0/30 dev vethA ... linkdown` (la ruta sigue, marcada `linkdown`) |
| hostA `ip neigh` | `10.10.1.2 dev vethA FAILED` |
| hostB `ip link` | `vethB` en `state DOWN` y sin el flag `UP` |
| hostB `ip addr` | `10.10.1.2/30` sigue asignada |
| hostB `ip route` | tabla vacía |

En la captura de `vethA` no apareció ninguna línea entre las 15:39:51 y las
15:43:05, es decir, durante la falla `vethA` no transmitió ARP ni ICMP.

### 8.4 Causa, corrección y recuperación

- **Causa:** `vethB` estaba administrativamente DOWN; `vethA` perdió el carrier
  y ARP no pudo resolver al vecino (`FAILED`). Las IP eran correctas: la falla
  es de enlace, no de direccionamiento.
- **Acción correctiva:** `sudo ip netns exec hostB ip link set vethB up`.
- **Prueba de recuperación:** ping de hostA a `10.10.1.2` con 4/4 recibidos y
  0 % de pérdida. La captura muestra además un nuevo ARP Request/Reply y los
  Echo Request/Reply (`id 1309`).

Contraste útil: en hostA (interfaz `UP` sin carrier) la ruta se conserva con
`linkdown`; en hostB (interfaz `DOWN`) la ruta no existe.

Evidencia: `evidencias/05-falla-diagnostico.txt`.

## 9. Uso de OpenCode

### Problema

ESCRIBE AQUÍ el problema real con el que usaste OpenCode.

### Consulta

ESCRIBE AQUÍ la consulta exacta que hiciste.

### Propuesta obtenida

ESCRIBE AQUÍ un resumen de lo que respondió.

### Verificación

ESCRIBE AQUÍ qué comando ejecutaste para comprobarlo y qué observaste.

### Explicación técnica

ESCRIBE AQUÍ, en tus palabras, por qué funciona o no lo propuesto.

## 10. Conclusiones

> Borrador: léelo, ajústalo y reescríbelo en tus palabras antes de entregar.

- Con `namespace + veth + IP + ruta` una sola computadora simula dos nodos de
  red independientes y se comunican con 0 % de pérdida.
- Un namespace aísla la pila de red pero comparte el kernel: no es una VM.
- Configurar una IP no activa la interfaz: hay que comprobar que exista, tenga
  dirección, esté `UP` y tenga carrier.
- La ruta conectada se crea sola cuando la interfaz está UP; en un enlace
  directo no hace falta gateway.
- La captura demuestra que ARP precede al ICMP y que, con la entrada en caché,
  no se repite.
- Ante la falla se diagnostica de abajo hacia arriba: la causa (`vethB` DOWN) se
  encontró comparando `ip link`, `ip route` e `ip neigh` en ambos extremos, sin
  reconstruir la topología.
- Esta base permite avanzar a routers con FRRouting, contenedores y BGP.
