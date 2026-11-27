# Laboratorio de Infraestructura y Soporte TI

**Proyecto práctico de virtualización, administración de servidores, servicios de red y gestión de recursos empresariales.**

## 1. Descripción del proyecto

Este proyecto consiste en la implementación de un laboratorio virtual de infraestructura tecnológica utilizando Hyper-V sobre Windows 11 Pro.

El entorno integra Windows Server y Ubuntu Server para practicar tareas de administración de servidores, gestión de identidades, servicios de red, acceso remoto, recursos compartidos y publicación de aplicaciones web.

El laboratorio simula componentes básicos de una infraestructura empresarial y permite comprobar la comunicación entre servidores y equipos clientes, así como la aplicación de controles de acceso a los recursos de red.

## 2. Objetivos

- Implementar un entorno virtualizado mediante Hyper-V.
- Instalar y configurar Windows Server y Ubuntu Server.
- Implementar un dominio mediante Active Directory.
- Administrar usuarios, grupos de seguridad y unidades organizativas.
- Configurar servicios DNS internos y resolución de nombres externos.
- Implementar DHCP para la asignación automática de direcciones IP.
- Configurar conectividad entre redes virtuales y acceso a Internet.
- Habilitar administración remota mediante SSH.
- Instalar Apache y publicar una página web personalizada.
- Crear recursos compartidos mediante SMB.
- Configurar y verificar permisos de acceso mediante SMB y NTFS.
- Documentar procedimientos, pruebas e incidencias técnicas.

## 3. Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| Windows 11 Pro | Sistema operativo anfitrión y cliente |
| Hyper-V | Plataforma de virtualización |
| Windows Server 2022 | Controlador de dominio y servidor de infraestructura |
| Ubuntu Server 26.04.1 LTS | Servidor Linux |
| Active Directory Domain Services | Administración del dominio |
| DNS | Resolución de nombres |
| DHCP | Asignación automática de direcciones IP |
| SMB | Compartición de archivos en red |
| NTFS | Administración de permisos sobre archivos y carpetas |
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

- Nombre de máquina virtual: `CLIENTE-W11-01`.
- Sistema operativo: Windows 11 Enterprise.
- Dirección IP obtenida mediante DHCP: `192.168.50.101`.
- Puerta de enlace: `192.168.50.1`.
- Servidor DNS: `192.168.50.10`.
- Sufijo DNS: `labred.local`.
- Dominio: `labred.local`.

El equipo cliente se encuentra unido al dominio y se utiliza para validar la autenticación de usuarios, las directivas de grupo, el acceso a recursos compartidos y la conectividad con los servidores Windows y Ubuntu.

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

Se creó la unidad organizativa principal `EmpresaLab`, con la siguiente estructura:

- `EmpresaLab`
  - `Administracion`
  - `SoporteTI`
  - `Ventas`
  - `Grupos`
  - `Equipos`

También se crearon tres usuarios de prueba y se ubicaron en sus respectivas unidades organizativas.

### 5.3. Administración de usuarios y grupos de seguridad

Se crearon los siguientes grupos globales de seguridad:

- `GG_Administracion`
- `GG_SoporteTI`
- `GG_Ventas`

Se crearon y asociaron los usuarios de prueba con sus respectivos grupos:

| Usuario | Departamento | Grupo |
|---|---|---|
| `agomez` | Administración | `GG_Administracion` |
| `cramirez` | Soporte TI | `GG_SoporteTI` |
| `lmartinez` | Ventas | `GG_Ventas` |

Se verificaron las ubicaciones de las cuentas en Active Directory y sus respectivas membresías de grupo.

### 5.4. Configuración de Ubuntu Server

Se implementó una máquina virtual Ubuntu Server con dirección IPv4 estática `192.168.50.20/24`.

Se configuró Netplan para establecer la dirección IP, la ruta predeterminada y el servidor DNS interno `192.168.50.10`.

Se verificó la conectividad con Windows Server y el acceso a Internet.

### 5.5. Servicios DNS

Se verificaron las zonas DNS `labred.local` y `_msdcs.labred.local`, así como registros A, NS, SOA y SRV.

Se configuraron reenviadores DNS en Windows Server utilizando `8.8.8.8` y `8.8.4.4`.

Se comprobó la resolución de nombres internos y externos desde Windows Server y Ubuntu.

Se creó un registro A para `srv-ubuntu01.labred.local`, asociado a `192.168.50.20`.

También se configuró el adaptador virtual del anfitrión para consultar el DNS interno al resolver nombres del dominio del laboratorio.

### 5.6. Servicio DHCP

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

### 5.7. Administración remota mediante SSH

Se habilitó el servicio SSH en Ubuntu Server y se comprobó que permaneciera activo.

Se verificó la escucha del servicio en el puerto TCP 22 y se estableció una conexión remota desde Windows 11 mediante el cliente SSH.

### 5.8. Implementación del servidor web Apache

Se instaló y habilitó Apache en Ubuntu Server.

Se creó una página web personalizada para identificar el laboratorio y se comprobó su funcionamiento desde el equipo anfitrión.

El acceso se verificó mediante la dirección IP `192.168.50.20` y el nombre DNS `srv-ubuntu01.labred.local`.

### 5.9. Recursos compartidos y permisos SMB/NTFS

Se crearon tres carpetas departamentales en Windows Server:

| Recurso compartido | Ruta local |
|---|---|
| `Administracion` | `C:\EmpresaLab\Administracion` |
| `SoporteTI` | `C:\EmpresaLab\SoporteTI` |
| `Ventas` | `C:\EmpresaLab\Ventas` |

Se configuraron permisos de recurso compartido SMB para conceder acceso de modificación al grupo de seguridad correspondiente a cada departamento y acceso completo a los administradores del dominio.

También se configuraron permisos NTFS explícitos en cada carpeta, protegiendo la herencia de permisos.

La configuración contempla:

- `SYSTEM`: control total.
- `BUILTIN\Administradores`: control total.
- `LABRED\Admins. del dominio`: control total.
- Grupo departamental correspondiente: modificación.

Se verificó que los usuarios pudieran acceder únicamente al recurso compartido autorizado para su departamento.

Las pruebas confirmaron que el acceso a las carpetas de otros departamentos era denegado.

### 5.10. Cliente Windows y pruebas de integración

Se configuró una máquina virtual cliente Windows 11 conectada al conmutador virtual `Switch-lab`.

Inicialmente se realizaron pruebas de conectividad y acceso a recursos compartidos desde un equipo perteneciente a `WORKGROUP`, utilizando credenciales del dominio.

Posteriormente, se incorporó el equipo `CLIENTE-W11-01` al dominio `labred.local`.

El cliente obtuvo mediante DHCP la dirección IPv4 `192.168.50.101`, con puerta de enlace `192.168.50.1` y servidor DNS `192.168.50.10`.

Se realizaron pruebas de conectividad con los servidores Windows y Ubuntu, resolución de nombres DNS, acceso a recursos compartidos y aplicación de directivas de grupo.

También se verificó el acceso autorizado a las carpetas departamentales y la denegación de acceso a recursos de otros departamentos.

### 5.11. Administración de recursos virtualizados

Debido a la disponibilidad de 16 GB de RAM del equipo anfitrión, las máquinas virtuales se administraron de manera escalonada.

Se realizaron pruebas de conectividad e integración encendiendo los servidores y el cliente según los servicios requeridos, sin necesidad de mantener toda la infraestructura activa simultáneamente.

### 5.12. Implementación de directivas de grupo (GPO)

Se configuraron directivas de grupo para centralizar políticas de seguridad y facilitar la administración de los equipos y usuarios del dominio.

Las actividades realizadas incluyen:

- Configuración de políticas de contraseñas y bloqueo de cuentas.
- Creación de directivas para el mapeo automático de unidades de red por departamento.
- Aplicación de configuraciones diferenciadas para los departamentos de Administración, Soporte TI y Ventas.
- Configuración de una política de seguridad para los equipos del laboratorio.
- Verificación de las directivas aplicadas mediante `gpupdate` y `gpresult`.

### 5.13. Integración DNS y conectividad entre servidores

Se integró Ubuntu Server con el servicio DNS del dominio `labred.local`.

Se realizaron las siguientes actividades:

- Creación del registro DNS A para `srv-ubuntu01.labred.local`.
- Configuración de la resolución inversa mediante registros PTR.
- Verificación de la resolución directa e inversa desde el cliente Windows.
- Comprobación de la conectividad con los servicios DNS, SSH y HTTP.
- Validación del acceso al servidor Apache mediante su nombre DNS.

### 5.14. Administración y mantenimiento de Ubuntu Server

Se realizaron actividades de administración preventiva y diagnóstico del servidor Linux.

Las actividades incluyen:

- Verificación del estado de los servicios SSH y Apache.
- Revisión de los puertos de escucha y registros del sistema.
- Consulta de actualizaciones disponibles mediante APT.
- Simulación de actualizaciones y limpieza de paquetes.
- Validación de la configuración de Apache mediante `apache2ctl configtest`.
- Verificación de la persistencia de la configuración de red después de reiniciar el servidor.

### 5.15. Respaldo y recuperación de configuraciones

Se realizó una práctica de respaldo y recuperación controlada de configuraciones del servidor Ubuntu.

El procedimiento incluyó:

- Verificación de la integridad del archivo comprimido.
- Inspección del contenido del respaldo.
- Extracción de los archivos en un directorio temporal.
- Comparación de la configuración de Netplan y archivos de prueba con sus originales.
- Validación de la recuperación sin sobrescribir las configuraciones activas del servidor.

### 5.16. Simulación y recuperación de una incidencia de Apache

Se realizó una simulación controlada de interrupción del servicio Apache en Ubuntu Server.

El ejercicio permitió practicar:

- Identificación del estado de un servicio detenido.
- Consulta de registros mediante `journalctl`.
- Validación de la configuración antes de iniciar el servicio.
- Recuperación del servicio Apache.
- Verificación del acceso HTTP desde el cliente Windows.
- Documentación de la causa, las acciones de recuperación y el resultado de la incidencia.

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
Get-ADUser -Filter * -Properties Department
Get-ADGroup -Filter *
Get-SmbShare
Get-SmbShareAccess
Get-Acl C:\EmpresaLab\Administracion
Get-Acl C:\EmpresaLab\SoporteTI
Get-Acl C:\EmpresaLab\Ventas
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

Las pruebas permitieron comprobar la conectividad entre equipos, la resolución de nombres, la asignación automática de direcciones IP, el acceso al servicio web y el funcionamiento de los controles de acceso a recursos compartidos.

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

- Ampliar los diagramas de arquitectura y documentación de los servicios implementados.
- Incorporar procedimientos de recuperación ante nuevas incidencias.
- Ampliar las pruebas de seguridad y administración de servidores.
- Evaluar nuevas integraciones entre Windows Server, Ubuntu Server y los equipos cliente.
- Fortalecer la documentación de las pruebas de respaldo y recuperación.

Las actividades de mantenimiento preventivo, diagnóstico de servicios Linux y recuperación controlada de configuraciones ya fueron implementadas y se encuentran documentadas en las carpetas del laboratorio.


## 9. Autoría

Proyecto personal de aprendizaje y práctica desarrollado para fortalecer competencias en Ingeniería de Sistemas, soporte TI, virtualización, administración de servidores y servicios de infraestructura tecnológica.

**Estado:** infraestructura principal implementada. Se han realizado pruebas de integración, administración de Active Directory, autenticación mediante credenciales de dominio y control de acceso a recursos compartidos SMB/NTFS.