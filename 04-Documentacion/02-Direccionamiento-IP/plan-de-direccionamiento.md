# Plan de direccionamiento IP

## 1. Información de la red

La red del laboratorio utiliza direccionamiento IPv4 privado y una máscara de subred `/24`.

| Parámetro | Valor |
|---|---|
| Dirección de red | 192.168.50.0 |
| Prefijo | /24 |
| Máscara | 255.255.255.0 |
| Dirección de broadcast | 192.168.50.255 |
| Puerta de enlace | 192.168.50.1 |
| Dominio interno | labred.local |

## 2. Direcciones IP asignadas

| Dispositivo | Dirección IP | Método de asignación |
|---|---|---|
| Adaptador virtual del anfitrión | 192.168.50.1 | Estática |
| SRV-DC01 | 192.168.50.10 | Estática |
| SRV-UBUNTU01 | 192.168.50.20 | Estática |
| CLIENTE-WIN11 | 192.168.50.100 | DHCP |

## 3. Configuración del servidor Windows

**Nombre:** SRV-DC01

- Dirección IPv4: `192.168.50.10`.
- Máscara: `255.255.255.0`.
- DNS: `127.0.0.1` y `::1`.
- Puerta de enlace: `192.168.50.1`.

El servidor utiliza una dirección estática porque proporciona servicios centrales de dominio, DNS y DHCP.

## 4. Configuración del servidor Ubuntu

**Nombre:** SRV-UBUNTU01

- Dirección IPv4: `192.168.50.20`.
- Prefijo: `/24`.
- Interfaz: `eth0`.
- Ruta predeterminada: `192.168.50.1`.
- Servidor DNS: `192.168.50.10`.
- Dominio de búsqueda: `labred.local`.

La configuración de red se administra mediante Netplan.

## 5. Ámbito DHCP

El servidor DHCP está instalado en `SRV-DC01`.

| Parámetro | Configuración |
|---|---|
| Nombre del ámbito | LAN-LABRED |
| Identificador de red | 192.168.50.0 |
| Máscara | 255.255.255.0 |
| Dirección inicial | 192.168.50.100 |
| Dirección final | 192.168.50.200 |
| Puerta de enlace | 192.168.50.1 |
| DNS | 192.168.50.10 |
| Dominio | labred.local |
| Duración de concesión | 8 días |

El ámbito contiene 101 direcciones disponibles para asignación dinámica.

Las direcciones de los servidores y del adaptador virtual del anfitrión se encuentran fuera del rango DHCP.

## 6. NAT y conectividad externa

Se creó en el equipo anfitrión una configuración NAT denominada `NAT-Laboratorio`, asociada al prefijo:

`192.168.50.0/24`

El siguiente salto utilizado por los servidores es `192.168.50.1`.

Esto permite que las máquinas virtuales accedan a recursos externos sin utilizar directamente la red física como su red de laboratorio.

## 7. Servidores DNS

El servidor DNS interno es `192.168.50.10`.

Se configuraron reenviadores externos:

- `8.8.8.8`
- `8.8.4.4`

Se creó un registro A para:

` srv-ubuntu01.labred.local `

Dirección asociada:

`192.168.50.20`

## 8. Verificaciones

Se comprobó el direccionamiento mediante:

```cmd
ipconfig /all
```

También se realizaron pruebas de conectividad:

```cmd
ping 192.168.50.10
ping 192.168.50.20
```

La asignación DHCP se verificó desde el cliente Windows, que recibió la dirección `192.168.50.100`.

## 9. Estado

**Implementado y probado:** direccionamiento estático de los servidores, red interna, NAT, DNS y asignación dinámica DHCP.