# Notas del Laboratorio 1: qué hace cada comando y por qué

## La idea general (en una frase)

Tienes **una sola computadora** y quieres simular **dos computadoras conectadas
por un cable**. Se logra con dos piezas:

- **namespace** = una "computadora falsa" (tiene sus propias interfaces, IP,
  rutas y tabla de vecinos, aislada de las demás).
- **veth** = un "cable de red virtual" con dos puntas. Lo que entra por una
  punta sale por la otra.

Analogía: el kernel es un edificio; cada namespace es una oficina cerrada con su
propio teléfono; el veth es un tubo que une dos oficinas.

```text
 hostA (oficina 1)                    hostB (oficina 2)
   vethA ═════════ cable veth ═════════ vethB
 10.10.1.1/30                         10.10.1.2/30
```

Fórmula del laboratorio: **namespace + veth + IP + interfaz UP = nodo de red**.

`sudo` = ejecutar como administrador (crear redes lo exige).

---

## Paso 1. Mirar la red antes de tocar nada

```bash
ip link      # lista las interfaces y su estado (UP/DOWN) y su MAC
ip addr      # muestra las direcciones IP de cada interfaz
ip route     # la tabla de rutas: por dónde sale cada paquete
ip neigh     # la tabla de vecinos: qué MAC corresponde a qué IP (ARP)
```

Se llaman "los cuatro comandos para mirar". Se usan **antes** (para ver el
estado) y **durante** (para diagnosticar). Piensa: link = cables y tarjetas,
addr = números de teléfono, route = mapa de caminos, neigh = agenda de
direcciones físicas.

```bash
ping -c 4 8.8.8.8      # envía 4 mensajes de prueba a 8.8.8.8 (-c = cuántos)
ping -c 4 google.com   # lo mismo pero con un nombre (usa DNS para traducirlo)
traceroute 8.8.8.8     # muestra por qué routers pasa el paquete
```

Si funciona la IP (8.8.8.8) pero no el nombre, el problema es el **DNS**.

---

## Paso 2. Crear los dos "computadores" (namespaces)

```bash
sudo ip netns add hostA     # crea el namespace llamado hostA
sudo ip netns add hostB     # crea el namespace llamado hostB
ip netns list               # lista los namespaces que existen
```

Un namespace recién creado está **vacío**: solo tiene `lo` (loopback) y está
apagado. No tiene rutas ni vecinos.

---

## Paso 3. Crear el cable (par veth)

```bash
sudo ip link add vethA type veth peer name vethB
#   add vethA      → crea una interfaz llamada vethA
#   type veth      → de tipo "cable virtual"
#   peer name vethB→ su otra punta se llama vethB
ip link            # ahora ves vethA y vethB en tu sistema principal
```

Las dos puntas **nacen juntas** y **en el mismo lugar** (tu sistema principal),
porque ahí ejecutaste el comando. Por eso las ves ahí al principio.

---

## Paso 4. Conectar cada punta a su computador

```bash
sudo ip link set vethA netns hostA   # mueve la punta vethA dentro de hostA
sudo ip link set vethB netns hostB   # mueve la punta vethB dentro de hostB
ip link                              # en el sistema principal ya NO aparecen
```

Una interfaz solo puede estar en **un** namespace a la vez; al moverla, desaparece
del origen. Por eso dejan de verse en el `ip link` principal.

```bash
sudo ip netns exec hostA ip link     # ejecuta "ip link" DENTRO de hostA
sudo ip netns exec hostB ip link     # ejecuta "ip link" DENTRO de hostB
```

`ip netns exec NOMBRE COMANDO` = "hazme entrar a esa computadora y ejecuta este
comando". **Todo lo que quieras hacer en hostA o hostB lleva ese prefijo.**

---

## Paso 5. Poner direcciones IP

```bash
sudo ip netns exec hostA ip addr add 10.10.1.1/30 dev vethA
sudo ip netns exec hostB ip addr add 10.10.1.2/30 dev vethB
#   addr add 10.10.1.1/30 → asigna esa IP
#   dev vethA             → a esa interfaz
```

Qué significa `/30`: de los 32 bits de la IP, 30 identifican la red y **2** son
para hosts. Con 2 bits hay 4 combinaciones: `.0` (red), `.1` y `.2` (hosts),
`.3` (broadcast). Solo `.1` y `.2` se pueden usar, que es justo lo que necesitan
dos extremos de un cable. Ambos están en la red `10.10.1.0/30`, por eso se ven
directamente.

```bash
sudo ip netns exec hostA ip addr     # verifica que la IP quedó asignada
sudo ip netns exec hostB ip addr
```

Ojo: poner la IP **no enciende** la interfaz. Sigue apagada (`DOWN`).

---

## Paso 6. Encender las interfaces

```bash
sudo ip netns exec hostA ip link set lo up       # enciende el loopback de hostA
sudo ip netns exec hostA ip link set vethA up    # enciende la punta vethA
sudo ip netns exec hostB ip link set lo up
sudo ip netns exec hostB ip link set vethB up
```

Una interfaz tiene que cumplir **cuatro** cosas para funcionar: existir, tener IP,
estar `UP` y tener *carrier* (señal). En un veth, el carrier aparece cuando
**las dos puntas** están UP.

| Palabra en `ip link` | Significado |
|---|---|
| `UP` | Encendida por el administrador |
| `DOWN` | Apagada |
| `LOWER_UP` | Hay señal (la otra punta está viva) |
| `NO-CARRIER` | No hay señal (la otra punta está apagada) |

---

## Paso 7. Ver las rutas

```bash
sudo ip netns exec hostA ip route
sudo ip netns exec hostB ip route
```

Salida esperada en hostA:

```text
10.10.1.0/30 dev vethA proto kernel scope link src 10.10.1.1
```

Cómo leerla:

| Parte | Significa |
|---|---|
| `10.10.1.0/30` | Para llegar a esta red... |
| `dev vethA` | ...salgo por la interfaz vethA |
| `proto kernel` | La ruta la creó el sistema solo (tú no la pusiste) |
| `scope link` | El destino está en el mismo cable: no hay router intermedio |
| `src 10.10.1.1` | Mi IP de origen será esta |

**No hay gateway** (no aparece `via ...`), porque un gateway sirve para llegar a
redes **lejanas**. Aquí el otro host está en el mismo cable.

---

## Paso 8. Probar que se ven (ping)

```bash
sudo ip netns exec hostA ping -c 4 10.10.1.2   # hostA llama a hostB
sudo ip netns exec hostB ping -c 4 10.10.1.1   # hostB llama a hostA
```

Si responde con `0% packet loss`, quedó probado de una vez: hay ruta, las
interfaces están UP, el cable funciona y la MAC del otro se resolvió.
Un `ttl=64` sin bajar indica que no pasó por ningún router.

---

## Paso 9. La tabla de vecinos (ARP)

```bash
sudo ip netns exec hostA ip neigh
sudo ip netns exec hostB ip neigh
```

Salida esperada:

```text
10.10.1.2 dev vethA lladdr 56:cc:ee:03:87:1a REACHABLE
```

**Por qué existe:** IP es la "dirección lógica", pero por el cable Ethernet los
datos se entregan con **MAC** (dirección física). hostA conoce la IP de hostB
pero no su MAC. **ARP** es la forma de preguntarlo:

1. hostA grita a todos (broadcast): "¿quién tiene 10.10.1.2?"
2. hostB responde solo a hostA: "yo, mi MAC es 56:cc:..."
3. hostA anota la respuesta en la tabla de vecinos (`ip neigh`).

`REACHABLE` = entrada confirmada hace poco. `STALE` = vieja, por revalidar.
`FAILED` = no se pudo resolver.

---

## Paso 10. Ver los paquetes de verdad (tcpdump)

Se usan **dos terminales**. Primero vacía la tabla de vecinos para que el ARP se
vuelva a ver:

```bash
sudo ip netns exec hostA ip neigh flush all    # borra la tabla de vecinos
sudo ip netns exec hostB ip neigh flush all
```

**Terminal 1** (queda escuchando):

```bash
sudo ip netns exec hostA tcpdump -n -e -i vethA arp or icmp
#   -n         → no traduce IPs a nombres (más rápido y claro)
#   -e         → muestra también las MAC
#   -i vethA   → escucha en esa interfaz
#   arp or icmp→ solo muestra ARP e ICMP (oculta el ruido de IPv6)
```

Parece "trabado", pero es normal: **espera tráfico**. Se detiene con `Ctrl+C`.

**Terminal 2** (genera tráfico):

```bash
sudo ip netns exec hostB ping -c 4 10.10.1.1
```

Lo que verás en la terminal 1, en este orden:

| # | Línea | Qué pasó |
|---|---|---|
| 1 | `ARP Request who-has 10.10.1.1 tell 10.10.1.2` | hostB pregunta la MAC (broadcast) |
| 2 | `ARP Reply 10.10.1.1 is-at d6:5b:...` | hostA responde con su MAC |
| 3 | `ICMP echo request` | el ping sale |
| 4 | `ICMP echo reply` | el ping vuelve |
| 5 | pares 3 y 4 repetidos | los demás pings, ya sin ARP |

Orden clave: **primero ARP, luego ICMP**.

Los mensajes `router solicitation` (IPv6) son ruido del sistema, no del ping.

---

## Paso 11. Romper la red a propósito (falla controlada)

Antes de ejecutar, **escribe tu predicción**. Luego:

```bash
sudo ip netns exec hostB ip link set vethB down   # apaga la punta de hostB
sudo ip netns exec hostA ping -c 4 10.10.1.2      # prueba desde hostA
```

Resultado: `Destination Host Unreachable` y 100 % de pérdida. Ahora diagnostica:

```bash
sudo ip netns exec hostA ip link     # vethA: NO-CARRIER (perdió señal)
sudo ip netns exec hostA ip addr     # la IP sigue ahí (no es problema de IP)
sudo ip netns exec hostA ip route    # ruta con "linkdown"
sudo ip netns exec hostA ip neigh    # vecino en FAILED (no se pudo resolver)
sudo ip netns exec hostB ip link     # vethB: DOWN → aquí está la causa
sudo ip netns exec hostB ip addr
sudo ip netns exec hostB ip route    # vacía: sin interfaz UP no hay ruta
```

Conclusión: la IP y la ruta eran correctas; la falla era de **enlace** (la
punta de hostB estaba apagada). Se diagnostica **de abajo hacia arriba**:
namespace → interfaz → UP → IP → ruta → vecino → paquetes.

Reparar y comprobar:

```bash
sudo ip netns exec hostB ip link set vethB up
sudo ip netns exec hostA ping -c 4 10.10.1.2      # vuelve a funcionar
```

---

## Paso 12. Limpiar (solo cuando el docente ya revisó)

```bash
sudo ip netns del hostA      # borra el namespace y sus interfaces
sudo ip netns del hostB
ip netns list               # no debe aparecer ninguno
```

Al borrar un namespace desaparece su punta del veth y, con ella, el par completo.
Los namespaces **no sobreviven a reiniciar WSL**.

---

## Paso 13. Guardar el trabajo con Git

```bash
cd ~/ETN1011        # entra al repositorio
git pull            # trae lo último del remoto
git status          # qué cambió y qué falta registrar
git add archivo     # prepara archivos para el commit
git commit -m "mensaje"   # registra una versión con una descripción
git log --oneline   # historial resumido de commits
git push            # publica tus commits en GitHub
```

Git = historial local. GitHub = copia remota. Un commit es una "foto" del
trabajo en un momento; el mensaje debe decir qué avance representa.

---

## Mini glosario

| Término | En una frase |
|---|---|
| Namespace | Computadora virtual aislada (solo su red) |
| veth | Cable Ethernet virtual de dos puntas |
| Interfaz | Tarjeta de red (física o virtual) |
| IP / prefijo `/30` | Dirección lógica y cuántos bits son de red |
| MAC | Dirección física de la interfaz |
| Ruta conectada | Ruta que el sistema crea solo al poner IP en una interfaz UP |
| Gateway | Router para llegar a redes lejanas (aquí no hace falta) |
| ARP | Pregunta "¿qué MAC tiene esta IP?" |
| ICMP | Protocolo de los `ping` |
| Carrier | Señal en el cable |
| tcpdump | Herramienta que muestra los paquetes que pasan |

## Resumen de una línea por paso

1. Mirar: `ip link/addr/route/neigh`
2. Crear PCs: `ip netns add`
3. Crear cable: `ip link add ... type veth`
4. Conectar: `ip link set ... netns`
5. IP: `ip addr add`
6. Encender: `ip link set ... up`
7. Rutas: `ip route`
8. Probar: `ping`
9. MAC de vecinos: `ip neigh`
10. Ver paquetes: `tcpdump`
11. Romper y arreglar: `ip link set ... down / up`
12. Limpiar: `ip netns del`