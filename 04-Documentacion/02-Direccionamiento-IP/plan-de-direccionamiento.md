# Plan de direccionamiento IP del laboratorio

## 1. Descripción de la red

El laboratorio utiliza una red IPv4 privada para interconectar las máquinas virtuales Windows Server, Ubuntu Server y Windows 11 cliente mediante el conmutador interno `Switch-lab` de Hyper-V.

El equipo anfitrión proporciona la puerta de enlace y la conectividad externa mediante NAT.

### Parámetros generales

| Parámetro | Configuración |
|---|---|
| Red | `192.168.50.0/24` |
| Máscara | `255.255.255.0` |
| Dirección de red | `192.168.50.0` |
| Dirección de difusión | `192.168.50.255` |
| Puerta de enlace | `192.168.50.1` |
| Servidor DNS | `192.168.50.10` |
| Dominio DNS | `labred.local` |
| Conmutador virtual | `Switch-lab` |
| Tipo de conmutador | Interno |
| NAT | `NAT-Laboratorio` |

La red dispone de 254 direcciones IPv4 de host utilizables. Las direcciones se distribuyen entre el anfitrión, los servidores y el ámbito DHCP.

## 2. Distribución de direcciones IPv4

| Equipo | Dirección IPv4 | Asignación | Función |
|---|---|---|---|
| Anfitrión Windows 11 Pro | `192.168.50.1` | Estática | Puerta de enlace, Hyper-V y NAT |
| `SRV-DC01` | `192.168.50.10` | Estática | AD DS, DNS, DHCP, SMB y GPO |
| `SRV-UBUNTU01` | `192.168.50.20` | Estática | Ubuntu Server, SSH y Apache |
| `CLIENTE-W11-01` | `192.168.50.101` | DHCP | Cliente Windows 11 unido al dominio |

**Nota:** la dirección `192.168.50.101` corresponde a la concesión DHCP observada durante las pruebas del cliente `CLIENTE-W11-01`. Puede cambiar si el servidor asigna otra dirección disponible.

## 3. Configuración del servidor Windows

El servidor `SRV-DC01` utiliza una dirección IPv4 estática para mantener disponibles los servicios de infraestructura.

### Parámetros de red

- Dirección IPv4: `192.168.50.10`.
- Máscara: `255.255.255.0`.
- Puerta de enlace: `192.168.50.1`.
- DNS preferido: `127.0.0.1`.
- Dominio: `labred.local`.
- Nombre NetBIOS: `LABRED`.

La dirección estática evita que los clientes y servidores dependan de una concesión DHCP para localizar los servicios del controlador de dominio.

## 4. Configuración del servidor Ubuntu

El servidor `SRV-UBUNTU01` utiliza una dirección IPv4 estática configurada mediante Netplan.

### Parámetros de red

- Interfaz: `eth0`.
- Dirección IPv4: `192.168.50.20/24`.
- Puerta de enlace: `192.168.50.1`.
- Servidor DNS: `192.168.50.10`.
- Dominio de búsqueda: `labred.local`.

La configuración se encuentra en:

`/etc/netplan/00-installer-config.yaml`

El servidor utiliza el DNS interno para resolver nombres del laboratorio y realizar consultas externas mediante los reenviadores configurados en Windows Server.

## 5. Configuración del servicio DHCP

El rol DHCP está instalado y autorizado en `SRV-DC01`.

### Ámbito LAN-LABRED

| Parámetro | Configuración |
|---|---|
| Identificador del ámbito | `192.168.50.0` |
| Rango inicial | `192.168.50.100` |
| Rango final | `192.168.50.200` |
| Máscara | `255.255.255.0` |
| Puerta de enlace | `192.168.50.1` |
| Servidor DNS | `192.168.50.10` |
| Dominio DNS | `labred.local` |
| Duración de concesión | 8 días |

El rango contiene 101 direcciones posibles antes de considerar concesiones activas, exclusiones u otras restricciones.

### Opciones DHCP

- Opción 003: Router — `192.168.50.1`.
- Opción 006: DNS Servers — `192.168.50.10`.
- Opción 015: DNS Domain Name — `labred.local`.

Se verificó la asignación automática de parámetros de red a los clientes Windows.

Inicialmente se realizaron pruebas con `CLIENTE-WIN11`, que obtuvo la dirección `192.168.50.100`. Posteriormente, el cliente `CLIENTE-W11-01`, unido al dominio, obtuvo la dirección `192.168.50.101`.

## 6. Configuración NAT y conectividad externa

El equipo anfitrión utiliza el adaptador virtual asociado a `Switch-lab` con la dirección `192.168.50.1`.

Se configuró el NAT `NAT-Laboratorio` para la red interna:

`192.168.50.0/24`

Las máquinas virtuales utilizan `192.168.50.1` como puerta de enlace para acceder a redes externas.

La conectividad externa depende de la conexión del anfitrión y de la configuración NAT.

## 7. Resolución DNS

El servidor Windows proporciona resolución de nombres internos para el dominio `labred.local`.

### Registros principales

| Nombre | Tipo | Dirección o destino |
|---|---|---|
| `srv-dc01.labred.local` | A | `192.168.50.10` |
| `srv-ubuntu01.labred.local` | A | `192.168.50.20` |
| `192.168.50.10` | PTR | `srv-dc01.labred.local` |
| `192.168.50.20` | PTR | `srv-ubuntu01.labred.local` |

Se configuró la zona de búsqueda inversa `50.168.192.in-addr.arpa` para resolver las direcciones de los servidores.

Los reenviadores DNS configurados en Windows Server son:

- `8.8.8.8`.
- `8.8.4.4`.

La resolución interna permite localizar los servidores por nombre, mientras que los reenviadores permiten resolver nombres externos.

## 8. Pruebas y verificaciones

### Desde Windows Server

```powershell
ipconfig /all
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.50.0
Get-DhcpServerv4Lease -ScopeId 192.168.50.0
Get-DnsServerZone
```

### Desde CLIENTE-W11-01

```powershell
ipconfig /all
ping 192.168.50.10
ping 192.168.50.20
nslookup srv-dc01.labred.local
nslookup srv-ubuntu01.labred.local
nslookup 192.168.50.10
nslookup 192.168.50.20
Test-NetConnection 192.168.50.20 -Port 22
Test-NetConnection 192.168.50.20 -Port 80
```

### Desde Ubuntu Server

```bash
ip addr
ip route
resolvectl status
resolvectl query srv-dc01.labred.local
resolvectl query google.com
```

## 9. Resultados

Las pruebas permitieron verificar:

- La comunicación entre los equipos de la red virtual.
- La asignación automática de direcciones mediante DHCP.
- La resolución DNS directa e inversa de los servidores.
- La conectividad TCP con los servicios SSH y Apache.
- El uso del servidor DNS interno por los equipos del laboratorio.

La dirección del cliente se considera dinámica, mientras que las direcciones del anfitrión y los servidores se mantienen estáticas.

## 10. Estado

El esquema de direccionamiento IPv4 está implementado y se han realizado pruebas de conectividad, resolución DNS y asignación DHCP.

Este documento sirve como referencia para la administración de la red, la incorporación de nuevos equipos y el diagnóstico de incidencias de conectividad.