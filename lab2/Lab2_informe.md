# Laboratorio: FRRouting en WSL/Ubuntu y equivalencias con Cisco IOS

| Campo              | Valor                                            |
| ------------------ | ------------------------------------------------ |
| Estudiante         | Josue Llanque                                    |
| Fecha de ejecución | 23 de octubre de 2026                            |
| Equipo (hostname)  | MSI                                              |
| Sistema            | Ubuntu 26.04.1 LTS (Resolute Raccoon) sobre WSL2 |
| Kernel             | 6.18.33.2-microsoft-standard-WSL2                |
| FRRouting          | 10.5.1 (paquete `10.5.1-1ubuntu4.1`)             |

> Las salidas de este informe son las obtenidas en mi equipo. Algunas están abreviadas con `[...]`
> (por ejemplo, la línea `configured with:` de `show version` y las listas repetidas de ayuda `?`).

---

## 1. Objetivo y alcance

Crear un router FRR dentro de Ubuntu sobre WSL2 y practicar con **un solo router**: interfaces, rutas conectadas,
rutas estáticas `blackhole`, persistencia con `write memory` y diagnóstico. Con un único router no puede formarse
una vecindad OSPF ni una sesión BGP; esos comandos se ejecutan solo para observar el comportamiento esperado.

## 2. Topología

```text
Windows
  └── WSL2 / Ubuntu
       └── R1: Linux + FRR
            ├── eth0       interfaz de WSL, no modificar su ruta por defecto
            └── dummy0     interfaz de laboratorio
                 10.10.10.1/24
```

Datos reales del entorno: `eth0` = `172.26.58.217/20`, gateway `172.26.48.1`. `dummy0` = `10.10.10.1/24`.

## 3. Fundamento teórico resumido

- **Plano de datos (kernel Linux):** mantiene la tabla de rutas instalada y reenvía paquetes.
- **Plano de control (FRR):** configuración, protocolos y selección de rutas. `zebra` coordina interfaces y rutas
  entre los daemons y el kernel (vía netlink); `staticd` gestiona rutas estáticas; `ospfd` y `bgpd` son opcionales.
- **Ruta conectada:** el kernel/FRR la crean al tener una IP en una interfaz activa (código `C`).
- **Ruta estática:** definida por el administrador (código `S`, distancia administrativa 1).
- **`blackhole` (Null0 en Cisco):** descarta los paquetes cuyo destino cae en el prefijo.
- **`write memory`:** guarda la configuración de FRR en `/etc/frr/frr.conf`; no guarda interfaces creadas con `ip link`.

---

## 4. Instalación de WSL y FRR

### Ejercicio 1. Comprobar WSL y Ubuntu

El comando `wsl --list --verbose` pertenece a **PowerShell** (Windows). Al ejecutarlo por error dentro de Ubuntu se obtuvo:

```text
jllanque@MSI:/mnt/c/WINDOWS/system32$ wsl --list --verbose
Command 'wsl' not found, but can be installed with:
sudo apt install wsl
```

Esto es esperado: el binario `wsl` no existe dentro de Linux. La comprobación en PowerShell:

```text
PEGAR SALIDA DE POWERSHELL (wsl --list --verbose)
```

Verificación dentro de Ubuntu:

```text
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
VERSION_CODENAME=resolute
ID=ubuntu
ID_LIKE=debian
[...]
$ uname -r
6.18.33.2-microsoft-standard-WSL2
$ whoami
jllanque
```

**Resultado:** Ubuntu activo y usuario Linux normal (`jllanque`).
**Cisco equivalente:** no existe; el sistema operativo de red ya viene instalado en el router.

### Ejercicio 2. Interfaces y rutas actuales

```text
$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 10.255.255.254/32 ::1/128
eth0             UP             172.26.58.217/20 fe80::215:5dff:fe27:4bf9/64
$ ip -4 route
default via 172.26.48.1 dev eth0 proto kernel
172.26.48.0/20 dev eth0 proto kernel scope link src 172.26.58.217
```

**Resultado:** interfaz de WSL = `eth0` con `172.26.58.217/20`; ruta predeterminada `default via 172.26.48.1 dev eth0`.
Esta ruta no se modificó durante el laboratorio.

**Cisco IOS equivalente:** `show interfaces`, `show ip interface brief`, `show ip route`.

### Ejercicio 3. Instalar FRR

```text
$ sudo apt install -y frr frr-pythontools iproute2 iputils-ping
[...]
Setting up frr (10.5.1-1ubuntu4.1) ...
Created symlink '/etc/systemd/system/multi-user.target.wants/frr.service' → '/usr/lib/systemd/system/frr.service'.
Setting up frr-pythontools (10.5.1-1ubuntu4.1) ...

$ apt policy frr
frr:
  Installed: 10.5.1-1ubuntu4.1
  Candidate: 10.5.1-1ubuntu4.1

$ ps -p 1 -o comm=
systemd

$ sudo systemctl status frr --no-pager
● frr.service - FRRouting
     Loaded: loaded (/usr/lib/systemd/system/frr.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-23 10:46:29 -04
     Status: "FRR Operational"
     CGroup: /system.slice/frr.service
             ├─1039 /usr/lib/frr/watchfrr -d mgmtd zebra staticd
             ├─1049 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─1051 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             └─1056 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1
```

**Resultado:** FRR 10.5.1 instalado; `systemd` ya estaba activo (no fue necesario editar `/etc/wsl.conf`
ni reiniciar WSL). El servicio quedó en `active (running)`.

**Cisco IOS equivalente:** la instalación corresponde al arranque del IOS y sus servicios; no se instalan BGP/OSPF como paquetes independientes.

---

## 5. Activar FRR y usar vtysh

### Ejercicio 4. `zebra` y `staticd`

Primero un incidente: sin `sudo`, el archivo no se puede leer.

```text
$ grep -E '^(zebra|staticd|ospfd|bgpd)=' /etc/frr/daemons
grep: /etc/frr/daemons: Permission denied
```

El archivo pertenece al usuario `frr` y solo `frr` o `root` pueden leerlo. Con `sudo`:

```text
$ sudo grep -E '^(zebra|staticd|ospfd|bgpd)=' /etc/frr/daemons
bgpd=no
ospfd=no
```

No aparecen `zebra=yes` ni `staticd=yes`. La razón está en el propio archivo:

```text
$ sudo grep -nE 'zebra|staticd|mgmtd' /etc/frr/daemons
15:# The watchfrr, zebra and staticd daemons are always started.
42:zebra_options="  -A 127.0.0.1 -s 90000000"
43:mgmtd_options="  -A 127.0.0.1"
58:staticd_options="-A 127.0.0.1"

$ ps -ef | grep -E 'zebra|staticd' | grep -v grep
root        1039       1  0 10:46 ?        00:00:00 /usr/lib/frr/watchfrr -d mgmtd zebra staticd
frr         1051       1  0 10:46 ?        00:00:00 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
frr         1056       1  0 10:46 ?        00:00:00 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1
```

**Conclusión:** en FRR 10.5.1 sobre Ubuntu 26.04, `watchfrr`, `zebra` y `staticd` (y `mgmtd`) se inician siempre;
`/etc/frr/daemons` solo habilita los daemons opcionales (`bgpd`, `ospfd`, etc.). No se agregaron las líneas manualmente.

Activación y comprobación del servicio:

```text
$ sudo systemctl enable --now frr
$ sudo systemctl restart frr
$ sudo systemctl status frr --no-pager
     Active: active (running) since Fri 2026-10-23 11:00:40 -04
     Status: "FRR Operational"
$ sudo vtysh -c 'show version'
FRRouting 10.5.1 (MSI) on Linux(6.18.33.2-microsoft-standard-WSL2).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with: [...]
$ sudo vtysh -c 'show ip route'
Codes: K - kernel route, C - connected, L - local, S - static, [...]

IPv4 unicast VRF default:
K>* 0.0.0.0/0 [0/0] via 172.26.48.1, eth0, weight 1, 00:00:13
L * 10.255.255.254/32 is directly connected, lo, weight 1, 00:00:13
C>* 10.255.255.254/32 is directly connected, lo, weight 1, 00:00:13
C>* 172.26.48.0/20 is directly connected, eth0, weight 1, 00:00:13
L>* 172.26.58.217/32 is directly connected, eth0, weight 1, 00:00:13
```

FRR aprendió las rutas de WSL a través de `zebra`: `K` (ruta por defecto del kernel), `C` (conectadas) y `L` (direcciones locales, que FRR 10 muestra aparte).

**Cisco IOS equivalente:** `show version`, `show ip route`.

### Ejercicio 5. Entrar a `vtysh` y explorar la CLI

```text
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
MSI# show version
FRRouting 10.5.1 (MSI) on Linux(6.18.33.2-microsoft-standard-WSL2).
[...]
MSI# show interface
Interface eth0 is up, line protocol is up
  Link ups:       6    last: 2026/10/23 11:03:52.17
  Link downs:     2    last: 2026/10/23 11:03:46.11
  flags: <UP,LOWER_UP,BROADCAST,RUNNING,MULTICAST>
  HWaddr: 00:15:5d:27:4b:f9
  inet 172.26.58.217/20
  [...]
Interface lo is up, line protocol is up
  inet 10.255.255.254/32
  [...]
MSI# show running-config
Current configuration:
!
frr version 10.5.1
frr defaults traditional
hostname MSI
domainname localdomain
log syslog informational
service integrated-vtysh-config
!
end
MSI# ?
  add, clear, configure, copy, debug, disable, enable, end, exit, find, list,
  mgmt, mtrace, no, output, ping, quit, rpki, send, show, terminal, test,
  traceroute, watchfrr, write            [lista abreviada]
MSI# exit
jllanque@MSI:~$
```

Observaciones:

- **Prompts:** Bash = `jllanque@MSI:~$`; FRR = `MSI#` (y `MSI(config)#` en configuración).
- `show interface` solo muestra `eth0` y `lo`; todavía no existe `dummy0`.
- `show running-config` está casi vacío (solo cabecera, hostname y `end`).
- `show ip route` muestra varias líneas `L ... inactive` repetidas de `172.26.58.217/32`. Son residuos de los cambios de estado de `eth0` en WSL (`Link downs: 2`); son inofensivas y no se modificaron.

**Cisco IOS equivalente:** `enable`, `show version`, `show interfaces`, `show ip route`, `show running-config`, `?`.

---

## 6. Interfaz de laboratorio

### Ejercicio 6. Crear y configurar `dummy0`

```text
$ sudo ip link add dummy0 type dummy
$ sudo ip addr add 10.10.10.1/24 dev dummy0
$ sudo ip link set dummy0 up
RTNETLINK answers: File exists
$ ip -br addr show dummy0
dummy0           UNKNOWN        10.10.10.1/24 fe80::8456:a6ff:fe24:5fbb/64
$ ip -4 route show 10.10.10.0/24
10.10.10.0/24 dev dummy0 proto kernel scope link src 10.10.10.1

$ ip link show dummy0
3: dummy0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 86:56:a6:24:5f:bb brd ff:ff:ff:ff:ff:ff
$ ip addr show dummy0
    inet 10.10.10.1/24 scope global dummy0
    inet6 fe80::8456:a6ff:fe24:5fbb/64 scope link proto kernel_ll
```

Uno de los comandos devolvió `RTNETLINK answers: File exists` (algo ya estaba creado), pero el estado final es el que pide la guía:
`dummy0` existe, está `UP` con estado `UNKNOWN` (normal en una interfaz dummy, que no tiene carrier físico), tiene `10.10.10.1/24`
y el kernel creó automáticamente la ruta conectada `10.10.10.0/24`.

Comprobación en FRR:

```text
$ sudo vtysh -c 'show interface dummy0'
Interface dummy0 is up, line protocol is up
  Link ups:       1    last: 2026/10/23 11:16:44.17
  Link downs:     1    last: 2026/10/23 11:16:44.16
  index 3 metric 0 mtu 1500 speed 0 txqlen 1000
  flags: <UP,LOWER_UP,BROADCAST,RUNNING,NOARP>
  Type: Ethernet
  HWaddr: 86:56:a6:24:5f:bb
  inet 10.10.10.1/24
  inet6 fe80::8456:a6ff:fe24:5fbb/64
  Interface Type dummy
```

FRR (zebra) detectó `dummy0` automáticamente vía netlink, aunque se creó con `ip link` y no con vtysh.

**Cisco IOS equivalente (conceptual):**

```text
configure terminal
interface GigabitEthernet0/1
 ip address 10.10.10.1 255.255.255.0
 no shutdown
end
show ip interface brief
```

---

## 7. Ejercicios con un solo router FRR

### Ejercicio 7. Ruta conectada

```text
$ ip -4 route show dev dummy0
10.10.10.0/24 proto kernel scope link src 10.10.10.1

$ sudo vtysh -c 'show ip route connected'
IPv4 unicast VRF default:
C>* 10.10.10.0/24 is directly connected, dummy0, weight 1, 00:07:45
C>* 10.255.255.254/32 is directly connected, lo, weight 1, 00:23:49
C>* 172.26.48.0/20 is directly connected, eth0, weight 1, 00:03:23
```

La ruta `10.10.10.0/24` aparece porque `dummy0` tiene una dirección dentro de esa subred. Es la misma ruta vista desde el kernel (`proto kernel`) y desde FRR (`C>*`).

**Cisco IOS:** `show ip route connected`, `show interfaces GigabitEthernet0/1`.

### Ejercicio 8. Ruta estática de descarte

```text
MSI# configure terminal
MSI(config)# ip route 198.51.100.0/24 blackhole
MSI(config)# end
MSI# show ip route 198.51.100.0/24
Routing entry for 198.51.100.0/24
  Known via "static", distance 1, metric 0, best
  Last update 00:00:24 ago
  Flags: Recursion Selected RR Distance
  Status: Installed
  * unreachable (blackhole), weight 1
```

`198.51.100.0/24` es un prefijo reservado para documentación (RFC 5737). La ruta es estática, con distancia administrativa 1, está instalada y descarta los paquetes.

**Cisco IOS equivalente aproximado:**

```text
configure terminal
ip route 198.51.100.0 255.255.255.0 Null0
end
show ip route 198.51.100.0
```

### Ejercicio 9. Confirmar desde Linux

```text
$ ip -4 route show 198.51.100.0/24
blackhole 198.51.100.0/24 proto static metric 20

$ sudo vtysh -c 'show ip route 198.51.100.0/24'
Routing entry for 198.51.100.0/24
  Known via "static", distance 1, metric 0, best
  Status: Installed
  * unreachable (blackhole), weight 1
```

| Vista          | Qué muestra                                                                                                                                    |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| FRR (control)  | Ruta estática (`static`), distancia 1, `unreachable (blackhole)`, estado `Installed`                                                           |
| Linux (kernel) | `blackhole 198.51.100.0/24 proto static metric 20`: ruta efectivamente instalada en el kernel; `proto static` identifica que la puso `staticd` |

### Ejercicio 10. Revisar la configuración activa

```text
MSI# show running-config
[...]
service integrated-vtysh-config
!
ip route 198.51.100.0/24 blackhole
!
end
MSI# show ip route static
IPv4 unicast VRF default:
S>* 198.51.100.0/24 [1/0] unreachable (blackhole), weight 1, 00:07:07
```

La configuración activa contiene la ruta estática; `show ip route static` filtra solo las rutas de código `S`.

### Ejercicio 11. Guardar la configuración

```text
MSI# write memory
Note: this version of vtysh never writes vtysh.conf
Building Configuration...
Integrated configuration saved to /etc/frr/frr.conf
[OK]

$ sudo grep -n "198.51.100.0" /etc/frr/frr.conf
8:ip route 198.51.100.0/24 blackhole
```

La ruta quedó guardada en `/etc/frr/frr.conf` (línea 8). El log de systemd también registra el guardado:
`watchfrr: configuration write completed with exit code 0` (11:40:12).

`write memory` guarda la configuración de FRR; **no** guarda `dummy0`, creada con `ip link`: esa interfaz no sobrevive a un reinicio de WSL.

**Cisco IOS equivalente:** `copy running-config startup-config`.

### Ejercicio 12. Eliminar y restaurar la ruta

```text
MSI# configure terminal
MSI(config)# no ip route 198.51.100.0/24 blackhole
MSI(config)# end
MSI# show ip route 198.51.100.0/24
% Network not in table

$ ip -4 route show 198.51.100.0/24
(sin salida)
```

La ruta desapareció tanto de FRR como del kernel. Restauración:

```text
MSI(config)# ip route 198.51.100.0/24 blackhole
MSI(config)# end
MSI# show ip route 198.51.100.0/24
Routing entry for 198.51.100.0/24
  Known via "static", distance 1, metric 0, best
  Status: Installed
  * unreachable (blackhole), weight 1
```

**Cisco IOS equivalente:** `no ip route 198.51.100.0 255.255.255.0 Null0`.

### Ejercicio 13. Probar el descarte

```text
$ ip route get 198.51.100.10
RTNETLINK answers: Invalid argument
$ ping -c 2 198.51.100.10
ping: connect: Invalid argument
```

El destino cae en la ruta `blackhole`. El kernel responde `Invalid argument` (EINVAL), que es el código que devuelve ante una ruta de descarte tipo blackhole: rechaza la consulta de ruta y el intento de conexión antes de enviar ningún paquete.
No hay respuesta que esperar; el objetivo era observar el descarte. (Para comparar: una ruta `unreachable` respondería `Network is unreachable` y una `prohibit`, `Permission denied`.)

**Cisco IOS equivalente:** `show ip route 198.51.100.10`, `ping 198.51.100.10`.

### Ejercicio 14. Estado de interfaces y contadores

```text
$ ip -s link show dummy0
3: dummy0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 86:56:a6:24:5f:bb brd ff:ff:ff:ff:ff:ff
    RX:  bytes packets errors dropped  missed   mcast
             0       0      0       0       0       0
    TX:  bytes packets errors dropped carrier collsns
           700      10      0       0       0       0
```

| Contador | Valor                               |
| -------- | ----------------------------------- |
| RX       | 0 bytes, 0 paquetes, sin errores    |
| TX       | 700 bytes, 10 paquetes, sin errores |

El TX distinto de cero corresponde probablemente a tráfico de control IPv6 generado por el propio kernel (no hubo pings, porque el blackhole los descartó). `show interface dummy0` en FRR confirma `line protocol is up`, `Interface Type dummy`, `inet 10.10.10.1/24`.

**Cisco IOS equivalente:** `show interfaces GigabitEthernet0/1`.

### Ejercicio 15. Diagnóstico con logs

```text
$ sudo systemctl status frr --no-pager
     Active: active (running) since Fri 2026-10-23 11:00:40 -04; 54min ago
     Status: "FRR Operational"

$ sudo journalctl -u frr -n 50 --no-pager     [extracto]
Oct 23 11:00:40 MSI zebra[1051]: Client 30 'static' (session id 0) encountered an error and is shutting down.
Oct 23 11:00:40 MSI zebra[1051]: Terminating on signal
Oct 23 11:00:40 MSI zebra[1051]: [...] Disabling MPLS support (no kernel support)
Oct 23 11:00:40 MSI watchfrr[1674]: mgmtd state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: zebra state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: staticd state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: all daemons up, doing startup-complete notify
Oct 23 11:40:12 MSI watchfrr[1674]: configuration write completed with exit code 0
```

Interpretación:

- El servicio está `active (running)` y todos los daemons subieron (`all daemons up`).
- `Disabling MPLS support (no kernel support)`: el kernel de WSL2 no incluye MPLS; no afecta al laboratorio.
- `Client 30 'static' ... shutting down`: aparece al detener el servicio durante el `restart` del ejercicio 4; es normal.
- No se registraron errores operativos.

Comandos en FRR: `show version`, `show running-config` (con la ruta blackhole) y `show ip route`:

```text
K>* 0.0.0.0/0 [0/0] via 172.26.48.1, eth0, weight 1, 00:55:59
C>* 10.10.10.0/24 is directly connected, dummy0, weight 1, 00:39:55
L>* 10.10.10.1/32 is directly connected, dummy0, weight 1, 00:39:55
C>* 10.255.255.254/32 is directly connected, lo, weight 1, 00:55:59
C>* 172.26.48.0/20 is directly connected, eth0, weight 1, 00:11:18
L>* 172.26.58.217/32 is directly connected, eth0, weight 1, 00:11:20
S>* 198.51.100.0/24 [1/0] unreachable (blackhole), weight 1, 00:09:41
[además, múltiples líneas "L ... eth0 inactive" repetidas, residuos de cambios de estado de eth0]
```

**Cisco IOS equivalente:** `show version`, `show running-config`, `show ip route`, `show logging`.

### Ejercicio 16. OSPF con un solo router

Antes de activar `ospfd`:

```text
MSI# show ip ospf neighbor
ospfd is not running
```

Activación del daemon:

```text
$ sudo sed -i 's/^ospfd=no/ospfd=yes/' /etc/frr/daemons
$ sudo systemctl restart frr
$ sudo systemctl status frr --no-pager
     Active: active (running) since Fri 2026-10-23 11:59:31 -04
     CGroup: /system.slice/frr.service
             ├─10164 /usr/lib/frr/watchfrr -d mgmtd zebra ospfd staticd
             ├─10175 /usr/lib/frr/mgmtd [...]
             ├─10177 /usr/lib/frr/zebra [...]
             ├─10182 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             └─10185 /usr/lib/frr/staticd [...]
```

Configuración y verificación:

```text
MSI# configure terminal
MSI(config)# router ospf
MSI(config-router)# network 10.10.10.0/24 area 0
MSI(config-router)# end
MSI# show ip ospf interface
dummy0 is up
  Internet Address 10.10.10.1/24, Broadcast 10.10.10.255, Area 0.0.0.0
  Router ID 10.255.255.254, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State Waiting, Priority 1
  No backup designated router on this network
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
  Neighbor Count is 0, Adjacent neighbor count is 0

MSI# show ip ospf neighbor

Neighbor ID     Pri State           Up Time         Dead Time Address         Interface        RXmtL RqstL DBsmL
(tabla vacía)

MSI# show ip ospf interface            [segunda consulta, tras el temporizador de espera]
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 10.255.255.254 Interface Address 10.10.10.1/24
  Neighbor Count is 0, Adjacent neighbor count is 0
```

Análisis:

- `dummy0` quedó en el área 0 (backbone), con Router ID `10.255.255.254` (tomado de `lo`), tipo `BROADCAST`, Hello 10 s y Dead 40 s.
- El estado pasó de `Waiting` a `DR` tras el _wait timer_ de 40 s: al no recibir Hellos de nadie, el router se eligió a sí mismo Designated Router.
- La tabla de vecinos quedó **vacía** (`Neighbor Count is 0`). Es el resultado esperado: OSPF necesita otro proceso OSPF conectado a la misma red para formar una vecindad.
- Quedó `ospfd=yes` en `/etc/frr/daemons`. La configuración de `router ospf` está en la configuración activa; no se guardó con `write memory` después del ejercicio 11.

**Cisco IOS equivalente:**

```text
configure terminal
router ospf 1
 network 10.10.10.0 0.0.0.255 area 0
end
show ip ospf interface
show ip ospf neighbor
```

---

## 8. Tabla rápida FRR frente a Cisco IOS

| Operación         | FRR / Linux                      | Cisco IOS clásico                    |
| ----------------- | -------------------------------- | ------------------------------------ |
| Ver interfaces    | `ip -br addr` / `show interface` | `show ip interface brief`            |
| Ver rutas         | `ip -4 route` / `show ip route`  | `show ip route`                      |
| Entrar a CLI      | `sudo vtysh`                     | `enable`                             |
| Configurar        | `configure terminal`             | `configure terminal`                 |
| IP de interfaz    | `ip addr add ... dev ...`        | `interface ...` + `ip address ...`   |
| Activar interfaz  | `ip link set ... up`             | `no shutdown`                        |
| Ruta estática     | `ip route PREFIJO NEXT_HOP`      | `ip route RED MÁSCARA NEXT_HOP`      |
| Ruta de descarte  | `blackhole`                      | `Null0`                              |
| Ver configuración | `show running-config`            | `show running-config`                |
| Guardar           | `write memory`                   | `copy running-config startup-config` |
| Logs              | `journalctl -u frr`              | `show logging`                       |
| OSPF vecinos      | `show ip ospf neighbor`          | `show ip ospf neighbor`              |

---

## 9. Preguntas de evaluación

1. **¿Qué diferencia hay entre FRR y el kernel Linux?**
   FRR es el plano de control: calcula y administra rutas y protocolos. El kernel es el plano de datos: mantiene la tabla instalada y reenvía los paquetes. En el laboratorio, FRR mostró la ruta como estática (`static`, distancia 1) y el kernel como `blackhole ... proto static metric 20`.
2. **¿Qué función cumple `zebra`?**
   Coordina interfaces y rutas entre los daemons de FRR y el kernel (vía netlink). Por eso detectó `dummy0` automáticamente y por eso instaló la ruta blackhole en Linux.
3. **¿Qué diferencia hay entre `show ip route` e `ip -4 route`?**
   `show ip route` es la vista de FRR (códigos `C`, `L`, `S`, `K`, distancia/métrica, estado `Installed`); `ip -4 route` es la tabla del kernel (`proto kernel`, `proto static`, `metric 20`). Son dos perspectivas de las mismas rutas.
4. **¿Qué conserva `write memory`?**
   Guarda la configuración de FRR en `/etc/frr/frr.conf` (verificado con `grep`: línea 8 con la ruta blackhole). No conserva las interfaces creadas con `ip link`, como `dummy0`.
5. **¿Por qué no aparecen vecinos OSPF?**
   Porque existe un único proceso OSPF; la vecindad requiere otro router OSPF en la misma red que intercambie mensajes Hello. Resultado observado: `Neighbor Count is 0` y tabla de vecinos vacía.
6. **¿Qué hace una ruta `blackhole`?**
   Descarta los paquetes destinados al prefijo. Observado: `ip route get` y `ping` devolvieron `Invalid argument`, sin enviar tráfico.
7. **¿Cuál es la equivalencia de `write memory` en Cisco IOS?**
   `copy running-config startup-config`.
8. **¿Por qué la IP se configura fuera de `vtysh` en este laboratorio?**
   Porque Linux administra directamente las interfaces y sus direcciones (`ip addr`), mientras que FRR administra rutas y protocolos. `zebra` solo aprende esos cambios a través de netlink (se comprobó con `dummy0`).

---

## 10. Conclusiones

- Se instaló y verificó FRR 10.5.1 en Ubuntu 26.04.1 sobre WSL2, con `systemd` activo y los daemons `mgmtd`, `zebra` y `staticd` corriendo.
- En esta versión, `zebra` y `staticd` se inician siempre; `/etc/frr/daemons` solo controla los daemons opcionales (`bgpd`, `ospfd`).
- FRR (control) y el kernel (datos) mantienen vistas distintas de la tabla de rutas, y coinciden: la ruta estática blackhole apareció en ambas.
- La interfaz `dummy0` creada con Linux fue detectada por `zebra` y generó una ruta conectada `10.10.10.0/24`.
- `write memory` guarda la configuración de FRR en `frr.conf`, pero no las interfaces del kernel.
- Una ruta `blackhole` hace que el kernel rechace el destino (`Invalid argument`) sin enviar paquetes.
- Con un solo router, OSPF se configura y la interfaz entra al área 0, pero no hay vecinos; para ver una adyacencia hace falta un segundo router (por ejemplo, otro namespace o contenedor con FRR).
- Cada comando de FRR tiene su equivalente aproximado en Cisco IOS, con diferencias en la forma de crear interfaces, asignar direcciones y guardar.

## 11. Limpieza

Estado al terminar la ejecución: `dummy0`, la ruta blackhole y la configuración de OSPF siguen activas, y `ospfd=yes` permanece en `/etc/frr/daemons`.
Comandos de limpieza previstos por la guía (no ejecutados todavía):

```bash
sudo ip link delete dummy0
```

```text
configure terminal
no ip route 198.51.100.0/24 blackhole
end
write memory
```

Opcionalmente, para dejar el equipo como al inicio: `no router ospf` en FRR y volver a `ospfd=no` en `/etc/frr/daemons`.
No se debe eliminar `eth0` ni la ruta `default` de WSL.

## Anexo. Código ejecutado en PowerShell

```powershell
jllanque@MSI:/mnt/c/WINDOWS/system32$ wsl --list --verbose
Command 'wsl' not found, but can be installed with:
sudo apt install wsl
jllanque@MSI:/mnt/c/WINDOWS/system32$ cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
VERSION_CODENAME=resolute
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=resolute
LOGO=ubuntu-logo
jllanque@MSI:/mnt/c/WINDOWS/system32$ uname -r
6.18.33.2-microsoft-standard-WSL2
jllanque@MSI:/mnt/c/WINDOWS/system32$ whoami
jllanque
jllanque@MSI:/mnt/c/WINDOWS/system32$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 10.255.255.254/32 ::1/128
eth0             UP             172.26.58.217/20 fe80::215:5dff:fe27:4bf9/64
jllanque@MSI:/mnt/c/WINDOWS/system32$ ip -4 route
default via 172.26.48.1 dev eth0 proto kernel
172.26.48.0/20 dev eth0 proto kernel scope link src 172.26.58.217
jllanque@MSI:/mnt/c/WINDOWS/system32$ sudo apt update
[sudo: authenticate] Password:
Get:1 http://security.ubuntu.com/ubuntu resolute-security InRelease [137 kB]
Hit:2 http://archive.ubuntu.com/ubuntu resolute InRelease
Get:3 http://archive.ubuntu.com/ubuntu resolute-updates InRelease [137 kB]
Get:4 http://security.ubuntu.com/ubuntu resolute-security/main amd64 Packages [606 kB]
Get:5 http://archive.ubuntu.com/ubuntu resolute-backports InRelease [137 kB]
Get:6 http://security.ubuntu.com/ubuntu resolute-security/main Translation-en [141 kB]
Get:7 http://security.ubuntu.com/ubuntu resolute-security/main amd64 Components [46.7 kB]
Get:8 http://security.ubuntu.com/ubuntu resolute-security/universe amd64 Packages [205 kB]
Get:9 http://security.ubuntu.com/ubuntu resolute-security/universe Translation-en [67.7 kB]
Get:10 http://security.ubuntu.com/ubuntu resolute-security/universe amd64 Components [55.6 kB]
Get:11 http://security.ubuntu.com/ubuntu resolute-security/restricted amd64 Packages [495 kB]
Get:12 http://security.ubuntu.com/ubuntu resolute-security/restricted Translation-en [99.6 kB]
Get:13 http://security.ubuntu.com/ubuntu resolute-security/multiverse amd64 Packages [20.0 kB]
Get:14 http://security.ubuntu.com/ubuntu resolute-security/multiverse Translation-en [3728 B]
Get:15 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 Packages [744 kB]
Get:16 http://archive.ubuntu.com/ubuntu resolute-updates/main Translation-en [176 kB]
Get:17 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 Components [110 kB]
Get:18 http://archive.ubuntu.com/ubuntu resolute-updates/universe amd64 Packages [334 kB]
Get:19 http://archive.ubuntu.com/ubuntu resolute-updates/universe Translation-en [110 kB]
Get:20 http://archive.ubuntu.com/ubuntu resolute-updates/universe amd64 Components [201 kB]
Get:21 http://archive.ubuntu.com/ubuntu resolute-updates/restricted amd64 Packages [508 kB]
Get:22 http://archive.ubuntu.com/ubuntu resolute-updates/restricted Translation-en [101 kB]
Get:23 http://archive.ubuntu.com/ubuntu resolute-updates/multiverse amd64 Packages [21.0 kB]
Get:24 http://archive.ubuntu.com/ubuntu resolute-updates/multiverse Translation-en [4168 B]
Get:25 http://archive.ubuntu.com/ubuntu resolute-backports/universe amd64 Packages [3136 B]
Get:26 http://archive.ubuntu.com/ubuntu resolute-backports/universe amd64 Components [1056 B]
Fetched 4466 kB in 4s (1137 kB/s)
59 packages can be upgraded. Run 'apt list --upgradable' to see them.
jllanque@MSI:/mnt/c/WINDOWS/system32$ sudo apt install -y frr frr-pythontools iproute2 iputils-ping
iproute2 is already the newest version (6.19.0-1ubuntu1.1).
iputils-ping is already the newest version (3:20250605-1ubuntu1).
Installing:
  frr  frr-pythontools

Installing dependencies:
  libcares2  liblua5.4-0  libunwind8  libyang3

Suggested packages:
  frr-doc

Summary:
  Upgrading: 0, Installing: 6, Removing: 0, Not Upgrading: 59
  Download size: 7466 kB
  Space needed: 42.7 MB / 1024 GB available

Get:1 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 libcares2 amd64 1.34.6-1ubuntu0.2 [102 kB]
Get:2 http://archive.ubuntu.com/ubuntu resolute/main amd64 liblua5.4-0 amd64 5.4.8-1build1 [163 kB]
Get:3 http://archive.ubuntu.com/ubuntu resolute/main amd64 libunwind8 amd64 1.8.3-0ubuntu1 [59.6 kB]
Get:4 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 libyang3 amd64 3.13.6-1ubuntu0.1 [533 kB]
Get:5 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 frr amd64 10.5.1-1ubuntu4.1 [6572 kB]
Get:6 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 frr-pythontools all 10.5.1-1ubuntu4.1 [37.2 kB]
Fetched 7466 kB in 3s (2544 kB/s)
Selecting previously unselected package libcares2:amd64.
(Reading database ... 36067 files and directories currently installed.)
Preparing to unpack .../0-libcares2_1.34.6-1ubuntu0.2_amd64.deb ...
Unpacking libcares2:amd64 (1.34.6-1ubuntu0.2) ...
Selecting previously unselected package liblua5.4-0:amd64.
Preparing to unpack .../1-liblua5.4-0_5.4.8-1build1_amd64.deb ...
Unpacking liblua5.4-0:amd64 (5.4.8-1build1) ...
Selecting previously unselected package libunwind8:amd64.
Preparing to unpack .../2-libunwind8_1.8.3-0ubuntu1_amd64.deb ...
Unpacking libunwind8:amd64 (1.8.3-0ubuntu1) ...
Selecting previously unselected package libyang3:amd64.
Preparing to unpack .../3-libyang3_3.13.6-1ubuntu0.1_amd64.deb ...
Unpacking libyang3:amd64 (3.13.6-1ubuntu0.1) ...
Selecting previously unselected package frr.
Preparing to unpack .../4-frr_10.5.1-1ubuntu4.1_amd64.deb ...
Unpacking frr (10.5.1-1ubuntu4.1) ...
Selecting previously unselected package frr-pythontools.
Preparing to unpack .../5-frr-pythontools_10.5.1-1ubuntu4.1_all.deb ...
Unpacking frr-pythontools (10.5.1-1ubuntu4.1) ...
Setting up libyang3:amd64 (3.13.6-1ubuntu0.1) ...
Setting up libunwind8:amd64 (1.8.3-0ubuntu1) ...
Setting up libcares2:amd64 (1.34.6-1ubuntu0.2) ...
Setting up liblua5.4-0:amd64 (5.4.8-1build1) ...
Setting up frr (10.5.1-1ubuntu4.1) ...
Created symlink '/etc/systemd/system/multi-user.target.wants/frr.service' → '/usr/lib/systemd/system/frr.service'.
Setting up frr-pythontools (10.5.1-1ubuntu4.1) ...
Processing triggers for rsyslog (8.2512.0-1ubuntu4.2) ...
Processing triggers for man-db (2.13.1-1build1) ...
Processing triggers for libc-bin (2.43-2ubuntu2.4) ...
ldconfig: /usr/lib/wsl/lib/libcuda.so.1 is not a symbolic link

jllanque@MSI:/mnt/c/WINDOWS/system32$ apt policy frr
frr:
  Installed: 10.5.1-1ubuntu4.1
  Candidate: 10.5.1-1ubuntu4.1
  Version table:
 *** 10.5.1-1ubuntu4.1 500
        500 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 Packages
        500 http://security.ubuntu.com/ubuntu resolute-security/main amd64 Packages
        100 /var/lib/dpkg/status
     10.5.1-1ubuntu4 500
        500 http://archive.ubuntu.com/ubuntu resolute/main amd64 Packages
jllanque@MSI:/mnt/c/WINDOWS/system32$ ps -p 1 -o comm=
systemd
jllanque@MSI:/mnt/c/WINDOWS/system32$ sudo systemctl status frr --no-pager
● frr.service - FRRouting
     Loaded: loaded (/usr/lib/systemd/system/frr.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-23 10:46:29 -04; 3min 18s ago
 Invocation: cd96c88910724d87a0c7acad9fe1eb08
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 1030 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 1039 (watchfrr)
     Status: "FRR Operational"
      Tasks: 8 (limit: 9335)
     Memory: 15.9M (peak: 31.7M)
        CPU: 210ms
     CGroup: /system.slice/frr.service
             ├─1039 /usr/lib/frr/watchfrr -d mgmtd zebra staticd
             ├─1049 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─1051 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             └─1056 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Oct 23 10:46:29 MSI frrinit.sh[1058]: Waiting for children to finish applying config...
Oct 23 10:46:29 MSI frrinit.sh[1075]: [1075|watchfrr] done
Oct 23 10:46:29 MSI staticd[1056]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Oct 23 10:46:29 MSI frrinit.sh[1077]: [1077|staticd] done
Oct 23 10:46:29 MSI watchfrr[1039]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Oct 23 10:46:29 MSI watchfrr[1039]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Oct 23 10:46:29 MSI watchfrr[1039]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Oct 23 10:46:29 MSI watchfrr[1039]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Oct 23 10:46:29 MSI frrinit.sh[1030]:  * Started watchfrr
Oct 23 10:46:29 MSI systemd[1]: Started frr.service - FRRouting.
jllanque@MSI:/mnt/c/WINDOWS/system32$ grep -E '^(zebra|staticd|ospfd|bgpd)=' /etc/frr/daemons
grep: /etc/frr/daemons: Permission denied
jllanque@MSI:/mnt/c/WINDOWS/system32$ sudo grep -E '^(zebra|staticd|ospfd|bgpd)=' /etc/frr/daemons
bgpd=no
ospfd=no
jllanque@MSI:/mnt/c/WINDOWS/system32$ sudo grep -nE 'zebra|staticd|mgmtd' /etc/frr/daemons
ps -ef | grep -E 'zebra|staticd' | grep -v grep
15:# The watchfrr, zebra and staticd daemons are always started.
42:zebra_options="  -A 127.0.0.1 -s 90000000"
43:mgmtd_options="  -A 127.0.0.1"
58:staticd_options="-A 127.0.0.1"
root        1039       1  0 10:46 ?        00:00:00 /usr/lib/frr/watchfrr -d mgmtd zebra staticd
frr         1051       1  0 10:46 ?        00:00:00 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
frr         1056       1  0 10:46 ?        00:00:00 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1
jllanque@MSI:/mnt/c/WINDOWS/system32$ sudo systemctl enable --now frr
sudo systemctl restart frr
sudo systemctl status frr --no-pager
● frr.service - FRRouting
     Loaded: loaded (/usr/lib/systemd/system/frr.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-23 11:00:40 -04; 14ms ago
 Invocation: 7c5812a420c54fa09a144f3ecb401998
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 1665 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 1674 (watchfrr)
     Status: "FRR Operational"
      Tasks: 8 (limit: 9335)
     Memory: 37.2M (peak: 49.6M)
        CPU: 125ms
     CGroup: /system.slice/frr.service
             ├─1674 /usr/lib/frr/watchfrr -d mgmtd zebra staticd
             ├─1684 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─1686 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             └─1691 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Oct 23 11:00:40 MSI watchfrr[1674]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Oct 23 11:00:40 MSI staticd[1691]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Oct 23 11:00:40 MSI frrinit.sh[1710]: [1710|watchfrr] done
Oct 23 11:00:40 MSI frrinit.sh[1712]: [1712|staticd] done
Oct 23 11:00:40 MSI watchfrr[1674]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Oct 23 11:00:40 MSI frrinit.sh[1665]:  * Started watchfrr
Oct 23 11:00:40 MSI systemd[1]: Started frr.service - FRRouting.
jllanque@MSI:/mnt/c/WINDOWS/system32$ sudo vtysh -c 'show version'
sudo vtysh -c 'show ip route'
FRRouting 10.5.1 (MSI) on Linux(6.18.33.2-microsoft-standard-WSL2).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--runstatedir=/run' '--disable-maintainer-mode' '--sbindir=/usr/lib/frr' '--with-vtysh-pager=/usr/bin/pager' '--libdir=/usr/lib/x86_64-linux-gnu/frr' '--with-moduledir=/usr/lib/x86_64-linux-gnu/frr/modules' '--disable-dependency-tracking' '--enable-rpki' '--enable-scripting' '--enable-pim6d' '--with-libpam' '--enable-doc' '--enable-doc-html' '--enable-snmp' '--enable-fpm' '--disable-protobuf' '--disable-zeromq' '--enable-ospfapi' '--enable-bgp-vnc' '--enable-multipath=256' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' 'build_alias=x86_64-linux-gnu' 'LIBS= -latomic' 'PYTHON=python3'
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
K>* 0.0.0.0/0 [0/0] via 172.26.48.1, eth0, weight 1, 00:00:13
L * 10.255.255.254/32 is directly connected, lo, weight 1, 00:00:13
C>* 10.255.255.254/32 is directly connected, lo, weight 1, 00:00:13
C>* 172.26.48.0/20 is directly connected, eth0, weight 1, 00:00:13
L>* 172.26.58.217/32 is directly connected, eth0, weight 1, 00:00:13
jllanque@MSI:/mnt/c/WINDOWS/system32$ cd ~
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# show version
FRRouting 10.5.1 (MSI) on Linux(6.18.33.2-microsoft-standard-WSL2).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--runstatedir=/run' '--disable-maintainer-mode' '--sbindir=/usr/lib/frr' '--with-vtysh-pager=/usr/bin/pager' '--libdir=/usr/lib/x86_64-linux-gnu/frr' '--with-moduledir=/usr/lib/x86_64-linux-gnu/frr/modules' '--disable-dependency-tracking' '--enable-rpki' '--enable-scripting' '--enable-pim6d' '--with-libpam' '--enable-doc' '--enable-doc-html' '--enable-snmp' '--enable-fpm' '--disable-protobuf' '--disable-zeromq' '--enable-ospfapi' '--enable-bgp-vnc' '--enable-multipath=256' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' 'build_alias=x86_64-linux-gnu' 'LIBS= -latomic' 'PYTHON=python3'
MSI# show interface
Interface eth0 is up, line protocol is up
  Link ups:       6    last: 2026/10/23 11:03:52.17
  Link downs:     2    last: 2026/10/23 11:03:46.11
  vrf: default
  index 2 metric 0 mtu 1500 speed 10000 txqlen 1000
  flags: <UP,LOWER_UP,BROADCAST,RUNNING,MULTICAST>
  MPLS  Not specified by CLI
  Multicast config is Not specified by CLI
  Shutdown config is Not specified by CLI
  Type: Ethernet
  HWaddr: 00:15:5d:27:4b:f9
  inet 172.26.58.217/20
  inet6 fe80::215:5dff:fe27:4bf9/64
  Interface Type Other
  Interface Slave Type None
  protodown: off
Interface lo is up, line protocol is up
  Link ups:       0    last: (never)
  Link downs:     0    last: (never)
  vrf: default
  index 1 metric 0 mtu 65536 speed 0 txqlen 1000
  flags: <UP,LOWER_UP,LOOPBACK,RUNNING>
  MPLS  Not specified by CLI
  Multicast config is Not specified by CLI
  Shutdown config is Not specified by CLI
  Type: Loopback
  inet 10.255.255.254/32
  Interface Type Other
  Interface Slave Type None
  protodown: off
MSI# show ip route
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
K>* 0.0.0.0/0 [0/0] via 172.26.48.1, eth0, weight 1, 00:05:06
L * 10.255.255.254/32 is directly connected, lo, weight 1, 00:05:06
C>* 10.255.255.254/32 is directly connected, lo, weight 1, 00:05:06
C>* 172.26.48.0/20 is directly connected, eth0, weight 1, 00:01:54
L * 172.26.58.217/32 is directly connected, eth0, weight 1, 00:01:54
L * 172.26.58.217/32 is directly connected, eth0, weight 1, 00:01:54
L>* 172.26.58.217/32 is directly connected, eth0, weight 1, 00:01:56
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:04:08
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:04:10
MSI# show running-config
Building configuration...

Current configuration:
!
frr version 10.5.1
frr defaults traditional
hostname MSI
domainname localdomain
log syslog informational
service integrated-vtysh-config
!
end
MSI#
  add               Add registration
  clear             Reset functions
  configure         Configuration from vty interface
  copy              Apply a configuration file
  debug             Debugging functions
  disable           Turn off privileged mode command
  enable            Turn on privileged mode command
  end               End current mode and change to enable mode
  exit              Exit current mode and down to previous mode
  find              Find CLI command matching a regular expression
  graceful-restart  Graceful Restart commands
  list              Print command list
  mgmt              Management Daemon (MGMTD) information
  mtrace            Multicast trace route to multicast source
  no                Negate a command or set its defaults
  output            Direct vtysh output to file
  ping              Send echo messages
  quit              Exit current mode and down to previous mode
  rpki              Control rpki specific settings
  send              Send message
  show              Show running system information
  terminal          Set terminal line parameters
  test              Test code
  traceroute        Trace route to destination
  watchfrr          Watchfrr Specific sub-command
  write             Write running configuration to memory, network, or terminal
MSI#
  add               Add registration
  clear             Reset functions
  configure         Configuration from vty interface
  copy              Apply a configuration file
  debug             Debugging functions
  disable           Turn off privileged mode command
  enable            Turn on privileged mode command
  end               End current mode and change to enable mode
  exit              Exit current mode and down to previous mode
  find              Find CLI command matching a regular expression
  graceful-restart  Graceful Restart commands
  list              Print command list
  mgmt              Management Daemon (MGMTD) information
  mtrace            Multicast trace route to multicast source
  no                Negate a command or set its defaults
  output            Direct vtysh output to file
  ping              Send echo messages
  quit              Exit current mode and down to previous mode
  rpki              Control rpki specific settings
  send              Send message
  show              Show running system information
  terminal          Set terminal line parameters
  test              Test code
  traceroute        Trace route to destination
  watchfrr          Watchfrr Specific sub-command
  write             Write running configuration to memory, network, or terminal
MSI#
  add               Add registration
  clear             Reset functions
  configure         Configuration from vty interface
  copy              Apply a configuration file
  debug             Debugging functions
  disable           Turn off privileged mode command
  enable            Turn on privileged mode command
  end               End current mode and change to enable mode
  exit              Exit current mode and down to previous mode
  find              Find CLI command matching a regular expression
  graceful-restart  Graceful Restart commands
  list              Print command list
  mgmt              Management Daemon (MGMTD) information
  mtrace            Multicast trace route to multicast source
  no                Negate a command or set its defaults
  output            Direct vtysh output to file
  ping              Send echo messages
  quit              Exit current mode and down to previous mode
  rpki              Control rpki specific settings
  send              Send message
  show              Show running system information
  terminal          Set terminal line parameters
  test              Test code
  traceroute        Trace route to destination
  watchfrr          Watchfrr Specific sub-command
  write             Write running configuration to memory, network, or terminal
MSI#
  add               Add registration
  clear             Reset functions
  configure         Configuration from vty interface
  copy              Apply a configuration file
  debug             Debugging functions
  disable           Turn off privileged mode command
  enable            Turn on privileged mode command
  end               End current mode and change to enable mode
  exit              Exit current mode and down to previous mode
  find              Find CLI command matching a regular expression
  graceful-restart  Graceful Restart commands
  list              Print command list
  mgmt              Management Daemon (MGMTD) information
  mtrace            Multicast trace route to multicast source
  no                Negate a command or set its defaults
  output            Direct vtysh output to file
  ping              Send echo messages
  quit              Exit current mode and down to previous mode
  rpki              Control rpki specific settings
  send              Send message
  show              Show running system information
  terminal          Set terminal line parameters
  test              Test code
  traceroute        Trace route to destination
  watchfrr          Watchfrr Specific sub-command
  write             Write running configuration to memory, network, or terminal
MSI# exit
jllanque@MSI:~$ sudo ip link add dummy0 type dummy
sudo ip addr add 10.10.10.1/24 dev dummy0
sudo ip link set dummy0 up
ip -br addr show dummy0
ip -4 route show 10.10.10.0/24
RTNETLINK answers: File exists
dummy0           UNKNOWN        10.10.10.1/24 fe80::8456:a6ff:fe24:5fbb/64
10.10.10.0/24 dev dummy0 proto kernel scope link src 10.10.10.1
jllanque@MSI:~$ ip link show dummy0
ip addr show dummy0
3: dummy0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 86:56:a6:24:5f:bb brd ff:ff:ff:ff:ff:ff
3: dummy0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 86:56:a6:24:5f:bb brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 scope global dummy0
       valid_lft forever preferred_lft forever
    inet6 fe80::8456:a6ff:fe24:5fbb/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
jllanque@MSI:~$ sudo vtysh -c 'show interface dummy0'
sudo vtysh -c 'show ip route connected'
Interface dummy0 is up, line protocol is up
  Link ups:       1    last: 2026/10/23 11:16:44.17
  Link downs:     1    last: 2026/10/23 11:16:44.16
  vrf: default
  index 3 metric 0 mtu 1500 speed 0 txqlen 1000
  flags: <UP,LOWER_UP,BROADCAST,RUNNING,NOARP>
  MPLS  Not specified by CLI
  Multicast config is Not specified by CLI
  Shutdown config is Not specified by CLI
  Type: Ethernet
  HWaddr: 86:56:a6:24:5f:bb
  inet 10.10.10.1/24
  inet6 fe80::8456:a6ff:fe24:5fbb/64
  Interface Type dummy
  Interface Slave Type None
  protodown: off
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
C>* 10.10.10.0/24 is directly connected, dummy0, weight 1, 00:06:34
C>* 10.255.255.254/32 is directly connected, lo, weight 1, 00:22:38
C>* 172.26.48.0/20 is directly connected, eth0, weight 1, 00:02:12
jllanque@MSI:~$ ip -4 route show dev dummy0
10.10.10.0/24 proto kernel scope link src 10.10.10.1
jllanque@MSI:~$ sudo vtysh -c 'show ip route connected'
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
C>* 10.10.10.0/24 is directly connected, dummy0, weight 1, 00:07:45
C>* 10.255.255.254/32 is directly connected, lo, weight 1, 00:23:49
C>* 172.26.48.0/20 is directly connected, eth0, weight 1, 00:03:23
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# configure terminal
MSI(config)# ip route 198.51.100.0/24 blackhole
MSI(config)# end
MSI# show ip route 198.51.100.0/24
Routing entry for 198.51.100.0/24
  Known via "static", distance 1, metric 0, best
  Last update 00:00:24 ago
  Flags: Recursion Selected RR Distance
  Status: Installed
  * unreachable (blackhole), weight 1

MSI# exit
jllanque@MSI:~$ ip -4 route show 198.51.100.0/24
blackhole 198.51.100.0/24 proto static metric 20
jllanque@MSI:~$ sudo vtysh -c 'show ip route 198.51.100.0/24'
Routing entry for 198.51.100.0/24
  Known via "static", distance 1, metric 0, best
  Last update 00:02:07 ago
  Flags: Recursion Selected RR Distance
  Status: Installed
  * unreachable (blackhole), weight 1

jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# show running-config
Building configuration...

Current configuration:
!
frr version 10.5.1
frr defaults traditional
hostname MSI
domainname localdomain
log syslog informational
service integrated-vtysh-config
!
ip route 198.51.100.0/24 blackhole
!
end
MSI# show ip route static
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
S>* 198.51.100.0/24 [1/0] unreachable (blackhole), weight 1, 00:07:07
MSI# exit
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# write memory
Note: this version of vtysh never writes vtysh.conf
Building Configuration...
Integrated configuration saved to /etc/frr/frr.conf
[OK]
MSI# exit
jllanque@MSI:~$ sudo grep -n "198.51.100.0" /etc/frr/frr.conf
8:ip route 198.51.100.0/24 blackhole
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# configure terminal
MSI(config)# no ip route 198.51.100.0/24 blackhole
MSI(config)# end
MSI# show ip route 198.51.100.0/24
% Network not in table
MSI# exit
jllanque@MSI:~$ ip -4 route show 198.51.100.0/24
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# configure terminal
MSI(config)# ip route 198.51.100.0/24 blackhole
MSI(config)# end
MSI# show ip route 198.51.100.0/24
Routing entry for 198.51.100.0/24
  Known via "static", distance 1, metric 0, best
  Last update 00:00:11 ago
  Flags: Recursion Selected RR Distance
  Status: Installed
  * unreachable (blackhole), weight 1

MSI# exit
jllanque@MSI:~$ ip route get 198.51.100.10
RTNETLINK answers: Invalid argument
jllanque@MSI:~$ ping -c 2 198.51.100.10
ping: connect: Invalid argument
jllanque@MSI:~$ ip -s link show dummy0
3: dummy0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 86:56:a6:24:5f:bb brd ff:ff:ff:ff:ff:ff
    RX:  bytes packets errors dropped  missed   mcast
             0       0      0       0       0       0
    TX:  bytes packets errors dropped carrier collsns
           700      10      0       0       0       0
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# show interface dummy0
Interface dummy0 is up, line protocol is up
  Link ups:       1    last: 2026/10/23 11:16:44.17
  Link downs:     1    last: 2026/10/23 11:16:44.16
  vrf: default
  index 3 metric 0 mtu 1500 speed 0 txqlen 1000
  flags: <UP,LOWER_UP,BROADCAST,RUNNING,NOARP>
  MPLS  Not specified by CLI
  Multicast config is Not specified by CLI
  Shutdown config is Not specified by CLI
  Type: Ethernet
  HWaddr: 86:56:a6:24:5f:bb
  inet 10.10.10.1/24
  inet6 fe80::8456:a6ff:fe24:5fbb/64
  Interface Type dummy
  Interface Slave Type None
  protodown: off
MSI# exit
jllanque@MSI:~$ sudo systemctl status frr --no-pager
sudo journalctl -u frr -n 50 --no-pager
● frr.service - FRRouting
     Loaded: loaded (/usr/lib/systemd/system/frr.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-23 11:00:40 -04; 54min ago
 Invocation: 7c5812a420c54fa09a144f3ecb401998
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
   Main PID: 1674 (watchfrr)
     Status: "FRR Operational"
      Tasks: 8 (limit: 9335)
     Memory: 26.1M (peak: 49.6M)
        CPU: 2.493s
     CGroup: /system.slice/frr.service
             ├─1674 /usr/lib/frr/watchfrr -d mgmtd zebra staticd
             ├─1684 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─1686 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             └─1691 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Oct 23 11:00:40 MSI staticd[1691]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Oct 23 11:00:40 MSI frrinit.sh[1710]: [1710|watchfrr] done
Oct 23 11:00:40 MSI frrinit.sh[1712]: [1712|staticd] done
Oct 23 11:00:40 MSI watchfrr[1674]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Oct 23 11:00:40 MSI frrinit.sh[1665]:  * Started watchfrr
Oct 23 11:00:40 MSI systemd[1]: Started frr.service - FRRouting.
Oct 23 11:40:12 MSI watchfrr[1674]: [WFP93-1D146] configuration write completed with exit code 0
Oct 23 10:46:29 MSI frrinit.sh[1077]: [1077|staticd] done
Oct 23 10:46:29 MSI watchfrr[1039]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Oct 23 10:46:29 MSI watchfrr[1039]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Oct 23 10:46:29 MSI watchfrr[1039]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Oct 23 10:46:29 MSI watchfrr[1039]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Oct 23 10:46:29 MSI frrinit.sh[1030]:  * Started watchfrr
Oct 23 10:46:29 MSI systemd[1]: Started frr.service - FRRouting.
Oct 23 11:00:40 MSI systemd[1]: Stopping frr.service - FRRouting...
Oct 23 11:00:40 MSI watchfrr[1039]: [NG1AJ-FP2TQ] Terminating on signal
Oct 23 11:00:40 MSI frrinit.sh[1622]:  * Stopped watchfrr
Oct 23 11:00:40 MSI staticd[1056]: [MRN6F-AYZC4] Terminating on signal
Oct 23 11:00:40 MSI zebra[1051]: [KW2G1-QNTQN][EC 4043309122] Client 30 'static' (session id 0) encountered an error and is shutting down.
Oct 23 11:00:40 MSI mgmtd[1049]: [X3G8F-PM93W] BE-adapter: mgmt_msg_read: got EOF/disconnect
Oct 23 11:00:40 MSI mgmtd[1049]: [J2RAS-MZ95C] Terminating on signal
Oct 23 11:00:40 MSI zebra[1051]: [XVBTQ-5QTVQ] Terminating on signal
Oct 23 11:00:40 MSI zebra[1051]: [KQB7H-NPVW9] ../zebra/zebra_ptm.c:1283 failed to find process pid registration
Oct 23 11:00:40 MSI zebra[1051]: [JPSA8-5KYEA] client 30 disconnected 0 static routes removed from the rib
Oct 23 11:00:40 MSI zebra[1051]: [S929C-NZR3N] client 30 disconnected 0 static nhgs removed from the rib
Oct 23 11:00:40 MSI zebra[1051]: [QS0NJ-H5QKJ] Zebra final shutdown
Oct 23 11:00:40 MSI frrinit.sh[1651]:  * Stopped staticd
Oct 23 11:00:40 MSI frrinit.sh[1652]:  * Stopped zebra
Oct 23 11:00:40 MSI frrinit.sh[1653]:  * Stopped mgmtd
Oct 23 11:00:40 MSI systemd[1]: frr.service: Deactivated successfully.
Oct 23 11:00:40 MSI systemd[1]: Stopped frr.service - FRRouting.
Oct 23 11:00:40 MSI systemd[1]: Starting frr.service - FRRouting...
Oct 23 11:00:40 MSI frrinit.sh[1665]:  * Starting watchfrr with command: '  /usr/lib/frr/watchfrr  -d   mgmtd zebra staticd'
Oct 23 11:00:40 MSI watchfrr[1674]: [T83RR-8SM5G] watchfrr 10.5.1 starting: vty@0
Oct 23 11:00:40 MSI watchfrr[1674]: [ZCJ3S-SPH5S] mgmtd state -> down : initial connection attempt failed
Oct 23 11:00:40 MSI watchfrr[1674]: [ZCJ3S-SPH5S] zebra state -> down : initial connection attempt failed
Oct 23 11:00:40 MSI watchfrr[1674]: [ZCJ3S-SPH5S] staticd state -> down : initial connection attempt failed
Oct 23 11:00:40 MSI watchfrr[1674]: [YFT0P-5Q5YX] Forked background command [pid 1675]: /usr/lib/frr/watchfrr.sh restart all
Oct 23 11:00:40 MSI frrinit.sh[1685]: 2026/10/23 11:00:40 ZEBRA: [KGY44-D47GD][EC 4043309111] Disabling MPLS support (no kernel support)
Oct 23 11:00:40 MSI frrinit.sh[1695]: [1695|mgmtd] sending configuration
Oct 23 11:00:40 MSI frrinit.sh[1696]: [1696|zebra] sending configuration
Oct 23 11:00:40 MSI frrinit.sh[1696]: [1696|zebra] done
Oct 23 11:00:40 MSI frrinit.sh[1710]: [1710|watchfrr] sending configuration
Oct 23 11:00:40 MSI frrinit.sh[1712]: [1712|staticd] sending configuration
Oct 23 11:00:40 MSI frrinit.sh[1695]: [1695|mgmtd] done
Oct 23 11:00:40 MSI frrinit.sh[1693]: Waiting for children to finish applying config...
Oct 23 11:00:40 MSI watchfrr[1674]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Oct 23 11:00:40 MSI staticd[1691]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Oct 23 11:00:40 MSI frrinit.sh[1710]: [1710|watchfrr] done
Oct 23 11:00:40 MSI frrinit.sh[1712]: [1712|staticd] done
Oct 23 11:00:40 MSI watchfrr[1674]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Oct 23 11:00:40 MSI watchfrr[1674]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Oct 23 11:00:40 MSI frrinit.sh[1665]:  * Started watchfrr
Oct 23 11:00:40 MSI systemd[1]: Started frr.service - FRRouting.
Oct 23 11:40:12 MSI watchfrr[1674]: [WFP93-1D146] configuration write completed with exit code 0
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# show version
FRRouting 10.5.1 (MSI) on Linux(6.18.33.2-microsoft-standard-WSL2).
Copyright 1996-2005 Kunihiro Ishiguro, et al.
configured with:
    '--build=x86_64-linux-gnu' '--prefix=/usr' '--includedir=${prefix}/include' '--mandir=${prefix}/share/man' '--infodir=${prefix}/share/info' '--sysconfdir=/etc' '--localstatedir=/var' '--disable-option-checking' '--disable-silent-rules' '--libdir=${prefix}/lib/x86_64-linux-gnu' '--runstatedir=/run' '--disable-maintainer-mode' '--sbindir=/usr/lib/frr' '--with-vtysh-pager=/usr/bin/pager' '--libdir=/usr/lib/x86_64-linux-gnu/frr' '--with-moduledir=/usr/lib/x86_64-linux-gnu/frr/modules' '--disable-dependency-tracking' '--enable-rpki' '--enable-scripting' '--enable-pim6d' '--with-libpam' '--enable-doc' '--enable-doc-html' '--enable-snmp' '--enable-fpm' '--disable-protobuf' '--disable-zeromq' '--enable-ospfapi' '--enable-bgp-vnc' '--enable-multipath=256' '--enable-user=frr' '--enable-group=frr' '--enable-vty-group=frrvty' '--enable-configfile-mask=0640' '--enable-logfile-mask=0640' 'build_alias=x86_64-linux-gnu' 'LIBS= -latomic' 'PYTHON=python3'
MSI# show running-config
Building configuration...

Current configuration:
!
frr version 10.5.1
frr defaults traditional
hostname MSI
domainname localdomain
log syslog informational
service integrated-vtysh-config
!
ip route 198.51.100.0/24 blackhole
!
end
MSI# show ip route
Codes: K - kernel route, C - connected, L - local, S - static,
       R - RIP, O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, F - PBR,
       f - OpenFabric, t - Table-Direct,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup
       t - trapped, o - offload failure

IPv4 unicast VRF default:
K>* 0.0.0.0/0 [0/0] via 172.26.48.1, eth0, weight 1, 00:55:59
C>* 10.10.10.0/24 is directly connected, dummy0, weight 1, 00:39:55
L>* 10.10.10.1/32 is directly connected, dummy0, weight 1, 00:39:55
L * 10.255.255.254/32 is directly connected, lo, weight 1, 00:55:59
C>* 10.255.255.254/32 is directly connected, lo, weight 1, 00:55:59
C>* 172.26.48.0/20 is directly connected, eth0, weight 1, 00:11:18
L * 172.26.58.217/32 is directly connected, eth0, weight 1, 00:11:18
L * 172.26.58.217/32 is directly connected, eth0, weight 1, 00:11:18
L>* 172.26.58.217/32 is directly connected, eth0, weight 1, 00:11:20
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:13:20
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:13:22
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:17:32
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:17:34
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:19:34
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:19:36
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:22:57
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:22:59
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:24:59
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:25:01
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:35:33
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:35:35
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:38:21
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:38:23
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:41:53
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:41:55
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:44:05
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:44:07
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:52:47
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:52:49
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:55:01
L   172.26.58.217/32 is directly connected, eth0 inactive, weight 1, 00:55:03
S>* 198.51.100.0/24 [1/0] unreachable (blackhole), weight 1, 00:09:41
MSI# exit
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# show ip ospf neighbor
ospfd is not running
MSI# exit
jllanque@MSI:~$ sudo sed -i 's/^ospfd=no/ospfd=yes/' /etc/frr/daemons
sudo systemctl restart frr
sudo systemctl status frr --no-pager
● frr.service - FRRouting
     Loaded: loaded (/usr/lib/systemd/system/frr.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-23 11:59:31 -04; 12ms ago
 Invocation: ed3b64bba2d64a2dbdf9f1fa7552b468
       Docs: https://frrouting.readthedocs.io/en/latest/setup.html
    Process: 10155 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
   Main PID: 10164 (watchfrr)
     Status: "FRR Operational"
      Tasks: 10 (limit: 9335)
     Memory: 26.7M (peak: 39M)
        CPU: 140ms
     CGroup: /system.slice/frr.service
             ├─10164 /usr/lib/frr/watchfrr -d mgmtd zebra ospfd staticd
             ├─10175 /usr/lib/frr/mgmtd -d -F traditional -A 127.0.0.1
             ├─10177 /usr/lib/frr/zebra -d -F traditional -A 127.0.0.1 -s 90000000
             ├─10182 /usr/lib/frr/ospfd -d -F traditional -A 127.0.0.1
             └─10185 /usr/lib/frr/staticd -d -F traditional -A 127.0.0.1

Oct 23 11:59:31 MSI staticd[10185]: [VTVCM-Y2NW3] Configuration Read in Took: 00:00:00
Oct 23 11:59:31 MSI frrinit.sh[10206]: [10206|staticd] done
Oct 23 11:59:31 MSI frrinit.sh[10187]: Waiting for children to finish applying config...
Oct 23 11:59:31 MSI watchfrr[10164]: [QDG3Y-BY5TN] mgmtd state -> up : connect succeeded
Oct 23 11:59:31 MSI watchfrr[10164]: [QDG3Y-BY5TN] zebra state -> up : connect succeeded
Oct 23 11:59:31 MSI watchfrr[10164]: [QDG3Y-BY5TN] ospfd state -> up : connect succeeded
Oct 23 11:59:31 MSI watchfrr[10164]: [QDG3Y-BY5TN] staticd state -> up : connect succeeded
Oct 23 11:59:31 MSI watchfrr[10164]: [KWE5Q-QNGFC] all daemons up, doing startup-complete notify
Oct 23 11:59:31 MSI frrinit.sh[10155]:  * Started watchfrr
Oct 23 11:59:31 MSI systemd[1]: Started frr.service - FRRouting.
jllanque@MSI:~$ sudo vtysh

Hello, this is FRRouting (version 10.5.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

MSI# configure terminal
MSI(config)# router ospf
MSI(config-router)# network 10.10.10.0/24 area 0
MSI(config-router)# end
MSI# show ip ospf interface
dummy0 is up
  ifindex 3, MTU 1500 bytes, BW 0 Mbit <UP,LOWER_UP,BROADCAST,RUNNING,NOARP>
  Internet Address 10.10.10.1/24, Broadcast 10.10.10.255, Area 0.0.0.0
  MTU mismatch detection: enabled
  Router ID 10.255.255.254, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State Waiting, Priority 1
  No backup designated router on this network
  Multicast group memberships: OSPFAllRouters
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
    Hello due in 9.341s
  Neighbor Count is 0, Adjacent neighbor count is 0
  Graceful Restart hello delay: 10s
  LSA retransmissions: 0

MSI# show ip opsf neighbor
% Unknown command: show ip opsf neighbor
MSI# show ospf neighbor
% Unknown command: show ospf neighbor
MSI# show ip ospf neighbor

Neighbor ID     Pri State           Up Time         Dead Time Address         Interface                        RXmtL RqstL DBsmL

MSI# show ip ospf interface
dummy0 is up
  ifindex 3, MTU 1500 bytes, BW 0 Mbit <UP,LOWER_UP,BROADCAST,RUNNING,NOARP>
  Internet Address 10.10.10.1/24, Broadcast 10.10.10.255, Area 0.0.0.0
  MTU mismatch detection: enabled
  Router ID 10.255.255.254, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 10.255.255.254 Interface Address 10.10.10.1/24
  No backup designated router on this network
  Multicast group memberships: OSPFAllRouters OSPFDesignatedRouters
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
    Hello due in 9.329s
  Neighbor Count is 0, Adjacent neighbor count is 0
  Graceful Restart hello delay: 10s
  LSA retransmissions: 0

MSI# show ip ospf neighbor

Neighbor ID     Pri State           Up Time         Dead Time Address         Interface                        RXmtL RqstL DBsmL

MSI# exit
jllanque@MSI:~$
```
