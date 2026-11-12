# Topología de red del laboratorio

## 1. Descripción general

El laboratorio utiliza Hyper-V sobre Windows 11 Pro para ejecutar tres máquinas virtuales conectadas mediante un conmutador virtual de tipo interno denominado `Switch-lab`.

La red permite la comunicación entre el equipo anfitrión, Windows Server, Ubuntu Server y el cliente Windows.

Se configuró NAT en el anfitrión para proporcionar conectividad hacia redes externas.

## 2. Componentes de la infraestructura

| Equipo | Nombre | Dirección IP | Función |
|---|---|---|---|
| Anfitrión | Windows 11 Pro | 192.168.50.1 | Virtualización y NAT |
| Servidor Windows | SRV-DC01 | 192.168.50.10 | Active Directory, DNS y DHCP |
| Servidor Linux | SRV-UBUNTU01 | 192.168.50.20 | Linux, SSH y Apache |
| Cliente | CLIENTE-WIN11 | 192.168.50.100 | Pruebas de servicios de red |

## 3. Diagrama lógico

```text
                 INTERNET
                    |
             Red del anfitrión
                    |
             Windows 11 Pro
                    |
           NAT-Laboratorio
                    |
        vEthernet (Switch-lab)
             192.168.50.1
                    |
             Switch-lab
             (Interno)
                    |
       +------------+-------------+
       |            |             |
       |            |             |
   SRV-DC01    SRV-UBUNTU01   CLIENTE-WIN11
   Windows       Ubuntu         Windows
   Server        Server         11
       |            |             |
  192.168.50.10 192.168.50.20 192.168.50.100
       |
  AD DS / DNS
     / DHCP
```

## 4. Funciones de los servidores

### SRV-DC01

Servidor principal del dominio `labred.local`.

Servicios implementados:

- Active Directory Domain Services.
- DNS.
- DHCP.
- Reenviadores DNS externos.

### SRV-UBUNTU01

Servidor Linux conectado a la red interna.

Servicios implementados:

- SSH para administración remota.
- Apache para publicación web.
- Resolución DNS mediante Windows Server.

### CLIENTE-WIN11

Equipo de pruebas conectado a la red virtual.

Funciones:

- Obtener una dirección IP mediante DHCP.
- Consultar el servidor DNS.
- Comprobar conectividad con los servidores.
- Acceder a la página web mediante nombre DNS.

## 5. Comunicación entre equipos

Se comprobaron las siguientes comunicaciones:

- Windows 11 anfitrión hacia `SRV-DC01`.
- Windows 11 anfitrión hacia `SRV-UBUNTU01`.
- `CLIENTE-WIN11` hacia `SRV-DC01`.
- `CLIENTE-WIN11` hacia `SRV-UBUNTU01`.
- Resolución del nombre `srv-ubuntu01.labred.local`.
- Acceso web mediante HTTP.

## 6. Consideraciones de virtualización

Las máquinas virtuales se conectan a `Switch-lab`, un conmutador interno de Hyper-V.

Debido a la memoria RAM disponible en el equipo anfitrión, las pruebas que requieren varios equipos se realizan encendiendo las máquinas virtuales por etapas cuando es necesario.

La infraestructura no requiere mantener todas las máquinas virtuales encendidas permanentemente.

## 7. Estado

**Implementado:** red virtual, comunicación entre equipos, conectividad externa mediante NAT y pruebas de acceso a los servicios configurados.