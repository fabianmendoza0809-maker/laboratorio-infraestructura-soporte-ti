# Topología de red del laboratorio

## 1. Descripción

El laboratorio utiliza una infraestructura virtualizada mediante Hyper-V sobre Windows 11 Pro. Las máquinas virtuales se conectan a un conmutador virtual interno denominado `Switch-lab`, dentro de la red `192.168.50.0/24`.

La arquitectura integra un controlador de dominio Windows Server, un servidor Ubuntu y un cliente Windows 11 unido al dominio.

El equipo anfitrión proporciona conectividad externa mediante NAT, mientras que Windows Server administra los servicios de identidad, DNS y DHCP del laboratorio.

## 2. Componentes de la infraestructura

| Equipo | Dirección IPv4 | Función |
|---|---|---|
| Anfitrión Windows 11 Pro | 192.168.50.1 | Hyper-V, conmutador virtual y NAT |
| SRV-DC01 | 192.168.50.10 | AD DS, DNS, DHCP, GPO y SMB |
| SRV-UBUNTU01 | 192.168.50.20 | Ubuntu Server, SSH y Apache |
| CLIENTE-W11-01 | 192.168.50.101 | Cliente Windows 11 unido al dominio, con DHCP |

La dirección del cliente corresponde a la concesión DHCP observada durante las pruebas. No es una dirección estática.

## 3. Esquema lógico

```text
                  Red externa / Internet
                           |
                  Equipo anfitrión
                  Windows 11 Pro
                           |
                  NAT-Laboratorio
                           |
              vEthernet 192.168.50.1
                           |
                      Switch-lab
                    192.168.50.0/24
                           |
          +----------------+----------------+
          |                |                |
      SRV-DC01        SRV-UBUNTU01     CLIENTE-W11-01
  192.168.50.10      192.168.50.20    192.168.50.101
          |                |                |
      AD DS / DNS       SSH / Apache      Dominio
      DHCP / SMB                          DHCP / GPO
      GPO
```

## 4. Configuración de red

- Red: `192.168.50.0/24`.
- Máscara: `255.255.255.0`.
- Conmutador virtual: `Switch-lab`, tipo interno.
- Puerta de enlace: `192.168.50.1`.
- NAT: `NAT-Laboratorio`.
- DNS interno: `192.168.50.10`.
- Dominio: `labred.local`.

## 5. Funciones de los equipos

### 5.1. SRV-DC01

Servidor Windows Server 2022 que proporciona:

- Active Directory Domain Services para el dominio `labred.local`.
- DNS para la resolución de nombres internos y externos.
- DHCP para la asignación automática de parámetros de red.
- Recursos compartidos SMB con permisos departamentales.
- Directivas de grupo para usuarios y equipos del dominio.

### 5.2. SRV-UBUNTU01

Servidor Ubuntu Server con dirección estática `192.168.50.20`.

Proporciona acceso remoto mediante SSH y publica una página web mediante Apache. Utiliza el DNS interno del laboratorio para resolver nombres del dominio y consultas externas.

### 5.3. CLIENTE-W11-01

Cliente Windows 11 Enterprise unido al dominio `labred.local`.

Se utiliza para comprobar:

- Asignación automática de parámetros de red mediante DHCP.
- Resolución DNS de los servidores.
- Conectividad con Windows Server y Ubuntu Server.
- Autenticación de usuarios del dominio.
- Acceso autorizado a recursos SMB departamentales.
- Aplicación de directivas de grupo y mapeo de unidades.
- Acceso al sitio web interno de Apache.

## 6. Comunicaciones y servicios

| Origen | Destino | Servicio |
|---|---|---|
| Cliente Windows | SRV-DC01 | DNS, DHCP, AD DS, SMB y GPO |
| Cliente Windows | SRV-UBUNTU01 | SSH y HTTP |
| SRV-UBUNTU01 | SRV-DC01 | Resolución DNS |
| Servidores y cliente | Anfitrión | Puerta de enlace y NAT |

## 7. Consideraciones de recursos

El equipo anfitrión dispone de 16 GB de RAM y un SSD de 512 GB.

Las máquinas virtuales se administran de manera escalonada según los servicios y las pruebas requeridas. Esta estrategia permite realizar prácticas de integración y diagnóstico sin mantener todos los equipos encendidos permanentemente.

## 8. Estado de la infraestructura

La infraestructura principal está implementada. Se han realizado pruebas de conectividad, resolución DNS, asignación DHCP, integración del cliente al dominio, acceso a recursos compartidos y aplicación de directivas de grupo.

La topología sirve como referencia para los procedimientos, las pruebas y el registro de incidencias del laboratorio.