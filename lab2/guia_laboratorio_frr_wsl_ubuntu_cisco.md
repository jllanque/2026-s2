# Guía de laboratorio: FRRouting en WSL/Ubuntu y equivalencias con Cisco IOS

## 1. Alcance

Esta guía crea un router FRR dentro de Ubuntu sobre WSL2. El laboratorio usa un único router, por lo que se practicarán interfaces, rutas estáticas, rutas conectadas, rutas `blackhole`, persistencia y diagnóstico.

Con un solo router no se puede formar una vecindad OSPF ni una sesión BGP real. Esos comandos aparecen únicamente como referencia de Cisco y como preparación para una ampliación con namespaces o un segundo router.

## 2. Topología

```text
Windows
  └── WSL2 / Ubuntu
       └── R1: Linux + FRR
            ├── eth0       interfaz de WSL, no modificar su ruta por defecto
            └── dummy0     interfaz de laboratorio
                 10.10.10.1/24
```

La interfaz `dummy0` permite practicar sin una segunda máquina. No representa una LAN externa real.

> Seguridad: trabaja en una instancia de práctica. No uses `ip route flush`, no borres la ruta `default` y no cambies `eth0` para resolver errores.

## 3. Convenciones

- **PowerShell:** comandos ejecutados en Windows.
- **Bash:** comandos ejecutados dentro de Ubuntu/WSL.
- **FRR:** comandos ejecutados dentro de `vtysh`.
- **Cisco IOS:** sintaxis equivalente aproximada. Las interfaces, permisos y persistencia no son iguales a FRR.

## 4. Instalación de WSL y FRR

### Ejercicio 1. Comprobar WSL y Ubuntu

En PowerShell:

```powershell
wsl --list --verbose
```

Si Ubuntu no está instalado:

```powershell
wsl --install -d Ubuntu
```

Inicia Ubuntu:

```powershell
wsl -d Ubuntu
```

En Ubuntu verifica el sistema:

```bash
cat /etc/os-release
uname -r
whoami
```

**Resultado esperado:** Ubuntu está activo y tienes un usuario Linux normal.

**Cisco equivalente:** no existe un equivalente directo. En Cisco, el sistema operativo de red ya viene instalado en el router.

### Ejercicio 2. Verificar interfaces y rutas actuales

En Bash:

```bash
ip -br addr
ip -4 route
```

Identifica la interfaz de WSL, normalmente `eth0`, y localiza la línea `default`.

**Resultado esperado:** ves la dirección IP de WSL y su ruta predeterminada.

**Cisco IOS equivalente:**

```text
show interfaces
show ip interface brief
show ip route
```

### Ejercicio 3. Instalar FRR

En Bash:

```bash
sudo apt update
sudo apt install -y frr frr-pythontools iproute2 iputils-ping
apt policy frr
```

Comprueba si `systemd` está activo:

```bash
ps -p 1 -o comm=
```

Si devuelve `systemd`:

```bash
sudo systemctl status frr --no-pager
```

Si no devuelve `systemd`, edita `/etc/wsl.conf`:

```ini
[boot]
systemd=true
```

Después, desde PowerShell:

```powershell
wsl --shutdown
wsl -d Ubuntu
```

**Cisco IOS equivalente:** la instalación de FRR corresponde al arranque del IOS y sus servicios. Normalmente no se instala BGP u OSPF como paquetes independientes.

## 5. Activar FRR y usar vtysh

### Ejercicio 4. Activar `zebra` y `staticd`

En Bash:

```bash
sudo nano /etc/frr/daemons
```

Asegura estas líneas, sin duplicarlas:

```ini
zebra=yes
staticd=yes
ospfd=no
bgpd=no
```

Activa el servicio:

```bash
sudo systemctl enable --now frr
sudo systemctl restart frr
sudo systemctl status frr --no-pager
```

Comprueba FRR sin entrar interactivamente:

```bash
sudo vtysh -c 'show version'
sudo vtysh -c 'show ip route'
```

**Cisco IOS equivalente:**

```text
show version
show ip route
```

### Ejercicio 5. Entrar a `vtysh` y explorar la CLI

En Bash:

```bash
sudo vtysh
```

En FRR:

```text
show version
show interface
show ip route
show running-config
?
exit
```

**Resultado esperado:** puedes distinguir el prompt de Bash del prompt de FRR.

**Cisco IOS equivalente:**

```text
enable
show version
show interfaces
show ip route
show running-config
?
```

## 6. Configurar un puerto de red de laboratorio

### Ejercicio 6. Crear y configurar `dummy0`

En Bash:

```bash
sudo ip link add dummy0 type dummy
sudo ip addr add 10.10.10.1/24 dev dummy0
sudo ip link set dummy0 up
ip -br addr show dummy0
ip -4 route show 10.10.10.0/24
```

Linux debe crear automáticamente la ruta conectada `10.10.10.0/24`.

Si aparece `RTNETLINK answers: File exists`, la interfaz ya existe. Compruébala con:

```bash
ip link show dummy0
ip addr show dummy0
```

**Cisco IOS equivalente:** en Cisco no se crea normalmente una interfaz `dummy` de esta forma. La operación conceptual sería:

```text
configure terminal
interface GigabitEthernet0/1
 ip address 10.10.10.1 255.255.255.0
 no shutdown
end
show ip interface brief
```

La dirección se configura en Linux con `ip addr`; en IOS se configura dentro del modo de interfaz.

## 7. Diez ejercicios con un solo router FRR

### Ejercicio 7. Consultar la ruta conectada

En Bash:

```bash
ip -4 route show dev dummy0
```

En FRR:

```bash
sudo vtysh -c 'show ip route connected'
```

La ruta conectada aparece porque `dummy0` tiene una dirección dentro de `10.10.10.0/24`.

**Cisco IOS:**

```text
show ip route connected
show interfaces GigabitEthernet0/1
```

### Ejercicio 8. Añadir una ruta estática de descarte

En FRR:

```text
sudo vtysh
configure terminal
ip route 198.51.100.0/24 blackhole
end
show ip route 198.51.100.0/24
```

`198.51.100.0/24` está reservado para documentación. La ruta descarta los paquetes.

**Cisco IOS equivalente aproximado:**

```text
configure terminal
ip route 198.51.100.0 255.255.255.0 Null0
end
show ip route 198.51.100.0
```

### Ejercicio 9. Confirmar la ruta desde Linux

En Bash:

```bash
ip -4 route show 198.51.100.0/24
sudo vtysh -c 'show ip route 198.51.100.0/24'
```

Compara ambas salidas. FRR muestra su perspectiva de control; Linux muestra la ruta instalada en el kernel.

**Cisco IOS:**

```text
show ip route 198.51.100.0
```

### Ejercicio 10. Revisar la configuración activa

En FRR:

```text
show running-config
show ip route static
```

**Cisco IOS equivalente:**

```text
show running-config
show ip route static
```

### Ejercicio 11. Guardar la configuración de FRR

En FRR:

```text
write memory
```

Comprueba el archivo desde Bash:

```bash
sudo grep -n "198.51.100.0" /etc/frr/frr.conf
```

**Cisco IOS equivalente:**

```text
copy running-config startup-config
```

En FRR, `write memory` guarda la configuración de FRR. No guarda automáticamente la interfaz creada con `ip link`.

### Ejercicio 12. Eliminar y restaurar la ruta

En FRR:

```text
configure terminal
no ip route 198.51.100.0/24 blackhole
end
show ip route 198.51.100.0/24
```

Restáurala:

```text
configure terminal
ip route 198.51.100.0/24 blackhole
end
```

**Cisco IOS equivalente:**

```text
configure terminal
no ip route 198.51.100.0 255.255.255.0 Null0
end
```

### Ejercicio 13. Probar el descarte con `ip route get`

En Bash:

```bash
ip route get 198.51.100.10
```

La salida debe indicar que el destino usa una ruta `blackhole` o de descarte.

Prueba también:

```bash
ping -c 2 198.51.100.10
```

No esperes una respuesta. El objetivo es observar el comportamiento de descarte.

**Cisco IOS equivalente:**

```text
show ip route 198.51.100.10
ping 198.51.100.10
```

### Ejercicio 14. Consultar el estado de interfaces y contadores

En Bash:

```bash
ip -s link show dummy0
```

En FRR:

```text
show interface dummy0
```

**Cisco IOS equivalente:**

```text
show interfaces GigabitEthernet0/1
```

### Ejercicio 15. Diagnosticar con logs

En Bash:

```bash
sudo systemctl status frr --no-pager
sudo journalctl -u frr -n 50 --no-pager
```

En FRR:

```text
show version
show running-config
show ip route
```

**Cisco IOS equivalente:**

```text
show version
show running-config
show ip route
show logging
```

### Ejercicio 16. Verificar qué ocurre con OSPF en un solo router

En FRR ejecuta:

```text
show ip ospf neighbor
```

No habrá vecinos porque `ospfd` está desactivado y no existe un segundo router.

Para dejar preparada la configuración, en Bash:

```bash
sudo sed -i 's/^ospfd=no/ospfd=yes/' /etc/frr/daemons
sudo systemctl restart frr
```

En FRR:

```text
configure terminal
router ospf
 network 10.10.10.0/24 area 0
end
show ip ospf interface
show ip ospf neighbor
```

**Cisco IOS equivalente:**

```text
configure terminal
router ospf 1
 network 10.10.10.0 0.0.0.255 area 0
end
show ip ospf interface
show ip ospf neighbor
```

El resultado esperado sigue siendo cero vecinos. OSPF necesita otro proceso OSPF conectado a la misma red.

## 8. Tabla rápida FRR frente a Cisco IOS

| Operación | FRR / Linux | Cisco IOS clásico |
|---|---|---|
| Ver interfaces | `ip -br addr` / `show interface` | `show ip interface brief` |
| Ver rutas | `ip -4 route` / `show ip route` | `show ip route` |
| Entrar a CLI | `sudo vtysh` | `enable` |
| Configurar | `configure terminal` | `configure terminal` |
| IP de interfaz | `ip addr add ... dev ...` | `interface ...` + `ip address ...` |
| Activar interfaz | `ip link set ... up` | `no shutdown` |
| Ruta estática | `ip route PREFIJO NEXT_HOP` | `ip route RED MÁSCARA NEXT_HOP` |
| Ruta de descarte | `blackhole` | `Null0` |
| Ver configuración | `show running-config` | `show running-config` |
| Guardar | `write memory` | `copy running-config startup-config` |
| Logs | `journalctl -u frr` | `show logging` |
| OSPF vecinos | `show ip ospf neighbor` | `show ip ospf neighbor` |

## 9. Limpieza

Si ya no necesitas la interfaz de laboratorio:

```bash
sudo ip link delete dummy0
```

Si deseas retirar la ruta desde FRR:

```text
configure terminal
no ip route 198.51.100.0/24 blackhole
end
write memory
```

No elimines `eth0` ni la ruta `default` de WSL.

## 10. Preguntas de evaluación

1. ¿Qué diferencia hay entre FRR y el kernel Linux?
2. ¿Qué función cumple `zebra`?
3. ¿Qué diferencia hay entre `show ip route` e `ip -4 route`?
4. ¿Qué conserva `write memory`?
5. ¿Por qué no aparecen vecinos OSPF?
6. ¿Qué hace una ruta `blackhole`?
7. ¿Cuál es la equivalencia de `write memory` en Cisco IOS?
8. ¿Por qué la configuración de una IP se hace fuera de `vtysh` en este laboratorio?

## 11. Respuestas breves

1. FRR calcula y administra rutas; Linux realiza el forwarding y mantiene la tabla del kernel.
2. `zebra` coordina interfaces y rutas entre los daemons FRR y el kernel.
3. `show ip route` muestra la vista de FRR; `ip -4 route` muestra la vista del kernel Linux.
4. Guarda la configuración de FRR, no las interfaces temporales creadas con `ip link`.
5. Porque no existe otro router OSPF conectado.
6. Descarta los paquetes destinados al prefijo.
7. `copy running-config startup-config`.
8. FRR administra protocolos y rutas, mientras Linux administra directamente las interfaces y sus direcciones.

## Referencias

- [Documentación oficial de FRRouting](https://docs.frrouting.org/en/latest/)
- [Configuración básica de FRR](https://docs.frrouting.org/en/latest/setup.html)
- [vtysh](https://docs.frrouting.org/en/latest/vtysh.html)
- [Rutas estáticas](https://docs.frrouting.org/en/latest/static.html)
- [Instalación de WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
