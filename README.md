# Laboratorio de Infraestructura y Soporte TI

**Proyecto práctico de virtualización, administración de servidores y servicios de red.**

## 1. Descripción del proyecto

Este proyecto consiste en la implementación de un laboratorio virtual de infraestructura tecnológica utilizando Hyper-V sobre Windows 11 Pro.

El entorno integra Windows Server y Ubuntu Server para practicar tareas de administración de servidores, gestión de identidades, servicios de red, acceso remoto y publicación de aplicaciones web.

El laboratorio simula componentes básicos de una infraestructura empresarial y permite comprobar la comunicación entre servidores y equipos clientes.

## 2. Objetivos

- Implementar un entorno virtualizado mediante Hyper-V.
- Instalar y configurar Windows Server y Ubuntu Server.
- Implementar un dominio mediante Active Directory.
- Administrar usuarios y unidades organizativas.
- Configurar servicios DNS internos y resolución de nombres externos.
- Implementar DHCP para la asignación automática de direcciones IP.
- Configurar conectividad entre redes virtuales y acceso a Internet.
- Habilitar administración remota mediante SSH.
- Instalar Apache y publicar una página web personalizada.
- Documentar procedimientos, pruebas e incidencias técnicas.

## 3. Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| Windows 11 Pro | Sistema operativo anfitrión |
| Hyper-V | Plataforma de virtualización |
| Windows Server 2022 | Controlador de dominio y servidor de infraestructura |
| Ubuntu Server | Servidor Linux - Sistema operativo: Ubuntu Server 26.04.1 LTS.|
| Active Directory Domain Services | Administración del dominio |
| DNS | Resolución de nombres |
| DHCP | Asignación automática de direcciones IP |
| Apache | Servicio web |
| SSH | Administración remota |
| PowerShell | Administración y verificación |
| Netplan | Configuración de red en Ubuntu |
| TCP/IP | Comunicación entre equipos |

## 4. Arquitectura del laboratorio

### Equipo anfitrión

- Sistema operativo: Windows 11 Pro.
- Plataforma de virtualización: Hyper-V.
- Memoria RAM física: 16 GB.
- Almacenamiento: SSD de 512 GB.

### Servidor Windows

- Nombre: `SRV-DC01`.
- Sistema operativo: Windows Server 2022 Standard Evaluation.
- Dirección IPv4: `192.168.50.10`.
- Máscara: `255.255.255.0`.
- Dominio: `labred.local`.
- Nombre NetBIOS: `LABRED`.
- Funciones: controlador de dominio, DNS y DHCP.

### Servidor Linux

- Nombre: `SRV-UBUNTU01`.
- Usuario administrativo: `adminlinux`.
- Dirección IPv4: `192.168.50.20`.
- Máscara: `/24`.
- Interfaz: `eth0`.
- DNS: `192.168.50.10`.
- Funciones: servidor Linux, acceso SSH y servicio web Apache.

### Cliente Windows

- Nombre de máquina virtual: `CLIENTE-WIN11`.
- Dirección IP obtenida mediante DHCP: `192.168.50.100`.
- Puerta de enlace: `192.168.50.1`.
- Servidor DNS: `192.168.50.10`.
- Sufijo DNS: `labred.local`.

### Red virtual

- Conmutador: `Switch-lab`.
- Tipo: Interno.
- Red: `192.168.50.0/24`.
- Adaptador virtual del anfitrión: `192.168.50.1`.
- NAT del anfitrión: `NAT-Laboratorio`.

## 5. Actividades implementadas

### 5.1. Virtualización e infraestructura de red

Se crearon máquinas virtuales mediante Hyper-V y se conectaron al conmutador interno `Switch-lab`.

Se configuró una red privada de laboratorio y un mecanismo NAT en el equipo anfitrión para proporcionar conectividad externa a las máquinas virtuales.

### 5.2. Windows Server y Active Directory

Se instaló Windows Server 2022 y se configuró como controlador de dominio.

Se creó un nuevo bosque con el dominio `labred.local` y el nombre NetBIOS `LABRED`.

Se habilitó DNS y se verificó el estado de los servicios principales del controlador de dominio.

También se creó la estructura organizativa:

- `EmpresaLab`
  - `Administracion`
  - `SoporteTI`
  - `Ventas`

Se creó el usuario de prueba `cramirez` dentro de la unidad organizativa `SoporteTI`.

### 5.3. Configuración de Ubuntu Server

Se implementó una máquina virtual Ubuntu Server con dirección IPv4 estática `192.168.50.20/24`.

Se configuró Netplan para establecer la dirección IP, la ruta predeterminada y el servidor DNS interno `192.168.50.10`.

Se verificó la conectividad con Windows Server y el acceso a Internet.

### 5.4. Servicios DNS

Se verificaron las zonas DNS `labred.local` y `_msdcs.labred.local`, así como registros A, NS, SOA y SRV.

Se configuraron reenviadores DNS en Windows Server utilizando `8.8.8.8` y `8.8.4.4`.

Se comprobó la resolución de nombres internos y externos desde Windows Server y Ubuntu.

Se creó un registro A para `srv-ubuntu01.labred.local`, asociado a `192.168.50.20`.

También se configuró el adaptador virtual del anfitrión para consultar el DNS interno al resolver nombres del dominio del laboratorio.

### 5.5. Servicio DHCP

Se instaló y autorizó el rol DHCP en Windows Server.

Se creó el ámbito `LAN-LABRED` con los siguientes parámetros:

- Red: `192.168.50.0/24`.
- Rango: `192.168.50.100` a `192.168.50.200`.
- Máscara: `255.255.255.0`.
- Puerta de enlace: `192.168.50.1`.
- Servidor DNS: `192.168.50.10`.
- Dominio: `labred.local`.
- Duración de concesión: 8 días.

Se comprobó la asignación automática de la dirección `192.168.50.100` al cliente Windows.

Las estadísticas del servidor confirmaron un intercambio DHCP exitoso: Discover, Offer, Request y ACK.

### 5.6. Administración remota mediante SSH

Se habilitó el servicio SSH en Ubuntu Server y se comprobó que permaneciera activo.

Se verificó la escucha del servicio en el puerto TCP 22 y se estableció una conexión remota desde Windows 11 mediante el cliente SSH.

### 5.7. Implementación del servidor web Apache

Se instaló y habilitó Apache en Ubuntu Server.

Se creó una página web personalizada para identificar el laboratorio y se comprobó su funcionamiento desde el equipo anfitrión.

El acceso se verificó mediante la dirección IP `192.168.50.20` y el nombre DNS `srv-ubuntu01.labred.local`.

### 5.8. Cliente Windows y pruebas de integración

Se creó una máquina virtual cliente conectada a `Switch-lab`.

Se configuró para obtener automáticamente su dirección IP y sus parámetros DNS.

Se verificó la asignación de `192.168.50.100`, la conectividad con Windows Server y Ubuntu, la resolución DNS y el acceso al sitio web publicado.

Debido a las limitaciones de memoria RAM del equipo anfitrión, algunas pruebas se realizaron encendiendo las máquinas virtuales por etapas.

### 5.9. Administración de recursos virtualizados

Debido a la disponibilidad de 16 GB de RAM del equipo anfitrión,
las máquinas virtuales se administraron de manera escalonada.

Se realizaron pruebas de conectividad e integración encendiendo
los servidores y el cliente según los servicios requeridos,
sin necesidad de mantener toda la infraestructura activa
simultáneamente.

## 6. Pruebas y verificaciones

Se utilizaron herramientas y comandos como:

**Windows Server / PowerShell:**

```powershell
hostname
ipconfig /all
Get-ADDomain
Get-ADForest
Get-Service DNS,NTDS
Get-DhcpServerInDC
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.50.0
Get-DhcpServerv4Statistics
Get-DhcpServerv4Lease -ScopeId 192.168.50.0
```

**Ubuntu Server:**

```bash
hostname
ip addr
ip route
resolvectl status
resolvectl query google.com
resolvectl query labred.local
systemctl status ssh
systemctl status apache2
```

**Cliente Windows:**

```cmd
ipconfig /all
ping 192.168.50.10
ping 192.168.50.20
nslookup srv-ubuntu01.labred.local
```

Las pruebas permitieron comprobar la conectividad entre equipos, la resolución de nombres, la asignación automática de direcciones IP y el acceso al servicio web.

## 7. Organización del repositorio

```text
laboratorio-infraestructura-soporte-ti/
│
├── README.md
├── 01-Windows-Server/
├── 02-Ubuntu-Server/
├── 03-DNS-DHCP/
└── 04-Documentacion/
    ├── README.md
    ├── 01-Arquitectura/
    ├── 02-Direccionamiento-IP/
    ├── 03-Procedimientos/
    ├── 04-Pruebas-y-Resultados/
    └── 05-Incidencias-y-Soluciones/
```

Las carpetas contienen evidencias de las configuraciones, procedimientos y verificaciones realizadas.

## 8. Actividades de mejora pendientes

- Crear grupos de seguridad en Active Directory y asociar usuarios.
- Configurar y probar políticas de grupo (GPO).
- Implementar y probar permisos de archivos y carpetas en Ubuntu.
- Ampliar la documentación técnica y los diagramas.
- Incorporar nuevas pruebas de administración y soporte de infraestructura.

## 9. Autoría

Proyecto personal de aprendizaje y práctica desarrollado para fortalecer competencias en Ingeniería de Sistemas, soporte TI, virtualización, administración de servidores y servicios de infraestructura tecnológica.

**Estado:** infraestructura principal implementada y pruebas de integración realizadas.