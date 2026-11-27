# Servicios configurados

## 1. Objetivo

Documentar los servicios implementados en el laboratorio de infraestructura, sus parámetros de configuración, las pruebas de funcionamiento y las evidencias disponibles.

El entorno integra Windows Server 2022, Ubuntu Server y un cliente Windows 11 mediante una red virtual interna de Hyper-V.

## 2. Resumen de servicios

| Servicio | Equipo | Configuración principal |
|---|---|---|
| Active Directory DS | SRV-DC01 | Dominio `labred.local` |
| DNS | SRV-DC01 | `192.168.50.10` |
| DHCP | SRV-DC01 | Ámbito `LAN-LABRED` |
| SMB | SRV-DC01 | Recursos departamentales |
| GPO | SRV-DC01 | Seguridad y unidades de red |
| SSH | SRV-UBUNTU01 | TCP 22 |
| Apache | SRV-UBUNTU01 | HTTP TCP 80 |
| NAT | Anfitrión Hyper-V | `NAT-Laboratorio` |

## 3. Active Directory Domain Services

### 3.1. Dominio

Se implementó un bosque de Active Directory con los siguientes parámetros:

- Servidor: `SRV-DC01`.
- Dominio DNS: `labred.local`.
- Nombre NetBIOS: `LABRED`.
- Nivel funcional del dominio: Windows Server 2016.
- Dirección IPv4: `192.168.50.10/24`.

### 3.2. Unidades organizativas

La estructura de la organización es:

```text
EmpresaLab
├── Administracion
├── SoporteTI
├── Ventas
├── Grupos
└── Equipos
```

### 3.3. Usuarios y grupos

Se crearon grupos globales de seguridad para representar los departamentos.

| Usuario | Departamento | Grupo |
|---|---|---|
| agomez | Administración | GG_Administracion |
| cramirez | Soporte TI | GG_SoporteTI |
| lmartinez | Ventas | GG_Ventas |

Los usuarios se ubicaron en sus unidades organizativas correspondientes y se asociaron a sus grupos de seguridad.

### 3.4. Verificación

Ejecutar en PowerShell con las herramientas de administración de Active Directory:

```powershell
Get-ADDomain
Get-ADForest
Get-ADUser -Filter * -Properties Department
Get-ADGroup -Filter *
Get-ADGroupMember GG_Administracion
Get-ADGroupMember GG_SoporteTI
Get-ADGroupMember GG_Ventas
Get-Service NTDS,DNS
```

Evidencias: `01-Windows-Server/03-Active-Directory/` y `01-Windows-Server/04-Usuarios-yGrupos/`.

## 4. Servicio DNS

### 4.1. Zonas y registros

El servidor DNS administra la zona `labred.local` y los registros necesarios para el funcionamiento del dominio.

Se verificaron:

- Zona directa `labred.local`.
- Zona `_msdcs.labred.local`.
- Registros A, NS, SOA y SRV.
- Registro A de `srv-ubuntu01.labred.local`, asociado a `192.168.50.20`.
- Zona inversa `50.168.192.in-addr.arpa`.
- Registros PTR para las direcciones `192.168.50.10` y `192.168.50.20`.

Se configuraron reenviadores DNS hacia `8.8.8.8` y `8.8.4.4` para la resolución de nombres externos.

### 4.2. Verificación

Desde Windows Server o el cliente:

```powershell
Resolve-DnsName srv-dc01.labred.local
Resolve-DnsName srv-ubuntu01.labred.local
nslookup 192.168.50.10
nslookup 192.168.50.20
```

Desde Ubuntu:

```bash
resolvectl status
resolvectl query srv-dc01.labred.local
resolvectl query srv-ubuntu01.labred.local
resolvectl query google.com
```

Evidencias: `03-DNS-DHCP/01-Configuracion-DNS/` y `03-DNS-DHCP/02-Registros-DNS/`.

## 5. Servicio DHCP

Se instaló y autorizó el rol DHCP en Windows Server.

### 5.1. Ámbito

| Parámetro | Valor |
|---|---|
| Nombre | LAN-LABRED |
| Red | 192.168.50.0/24 |
| Rango | 192.168.50.100 - 192.168.50.200 |
| Máscara | 255.255.255.0 |
| Puerta de enlace | 192.168.50.1 |
| DNS | 192.168.50.10 |
| Dominio DNS | labred.local |
| Concesión | 8 días |

El cliente inicial `CLIENTE-WIN11` obtuvo la dirección `192.168.50.100`. Posteriormente, el cliente integrado al dominio `CLIENTE-W11-01` obtuvo `192.168.50.101`.

### 5.2. Verificación

```powershell
Get-DhcpServerInDC
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.50.0
Get-DhcpServerv4Statistics
Get-DhcpServerv4Lease -ScopeId 192.168.50.0
```

En el cliente:

```cmd
ipconfig /all
```

Se verificó la entrega de dirección IP, puerta de enlace, DNS y sufijo de dominio.

Evidencias: `03-DNS-DHCP/03-Configuracion-DHCP/` y `03-DNS-DHCP/04-Pruebas/`.

## 6. Recursos compartidos SMB y permisos NTFS

### 6.1. Recursos

| Recurso SMB | Ruta local | Grupo autorizado |
|---|---|---|
| Administracion | C:\EmpresaLab\Administracion | GG_Administracion |
| SoporteTI | C:\EmpresaLab\SoporteTI | GG_SoporteTI |
| Ventas | C:\EmpresaLab\Ventas | GG_Ventas |

### 6.2. Permisos

En cada recurso compartido se concedió:

- Control total a los administradores del dominio.
- Cambio al grupo departamental correspondiente.

En NTFS se configuraron permisos explícitos y se protegió la herencia de permisos:

- `SYSTEM`: control total.
- `BUILTIN\Administradores`: control total.
- `LABRED\Admins. del dominio`: control total.
- Grupo departamental correspondiente: modificación.

Se realizaron pruebas de acceso autorizado y de denegación de acceso a recursos de otros departamentos.

### 6.3. Verificación

```powershell
Get-SmbShare
Get-SmbShareAccess -Name Administracion
Get-SmbShareAccess -Name SoporteTI
Get-SmbShareAccess -Name Ventas
Get-Acl C:\EmpresaLab\Administracion
Get-Acl C:\EmpresaLab\SoporteTI
Get-Acl C:\EmpresaLab\Ventas
```

Desde el cliente se pueden comprobar las rutas:

```text
\\SRV-DC01\Administracion
\\SRV-DC01\SoporteTI
\\SRV-DC01\Ventas
```

Evidencias: `01-Windows-Server/05-Recursos-Compartidos-y-Permisos/`.

## 7. Integración del cliente Windows y GPO

### 7.1. Cliente de dominio

Se incorporó `CLIENTE-W11-01` al dominio `labred.local` y se ubicó su cuenta de equipo en la unidad organizativa `Equipos`.

El cliente utiliza DHCP y el DNS interno del controlador de dominio.

### 7.2. Directivas de grupo

Se configuraron las siguientes GPO:

| GPO | Propósito |
|---|---|
| GPO-Politica-Contrasenas-Dominio | Política de contraseñas y bloqueo |
| GPO-Mapeo-Unidad-Administracion | Unidad A: |
| GPO-Mapeo-Unidad-SoporteTI | Unidad S: |
| GPO-Mapeo-Unidad-Ventas | Unidad V: |
| GPO-Seguridad-Equipos | Inactividad de equipos |

Las unidades se asignaron según el departamento y se configuraron para reconectarse al iniciar sesión.

La directiva de seguridad de equipos estableció un tiempo de inactividad de 600 segundos.

### 7.3. Política de contraseñas

- Longitud mínima: 8 caracteres.
- Complejidad habilitada.
- Historial: 5 contraseñas.
- Edad mínima: 1 día.
- Edad máxima: 90 días.
- Umbral de bloqueo: 5 intentos.
- Duración del bloqueo: 15 minutos.
- Ventana de observación: 15 minutos.
- Almacenamiento reversible: deshabilitado.

### 7.4. Verificación

Desde el cliente:

```cmd
gpupdate /force
gpresult /r
whoami /groups
```

Desde Windows Server:

```powershell
Get-ADComputer CLIENTE-W11-01 -Properties DistinguishedName
Get-GPO -All
Get-ADDefaultDomainPasswordPolicy
```

Se verificó la aplicación de las directivas y el acceso a las unidades departamentales.

Evidencias: carpetas `06-Integracion-Cliente-Dominio` a `10-Seguridad-Equipos-GPO` dentro de `01-Windows-Server/`.

## 8. Administración remota mediante SSH

En `SRV-UBUNTU01` se habilitó OpenSSH Server para la administración remota desde Windows.

- Dirección: `192.168.50.20`.
- Puerto TCP: 22.
- Servicio: `ssh`.
- Usuario administrativo: `adminlinux`.

### Verificación

```bash
systemctl status ssh
systemctl is-enabled ssh
sudo ss -tlnp
```

Desde Windows:

```powershell
Test-NetConnection srv-ubuntu01.labred.local -Port 22
ssh adminlinux@192.168.50.20
```

Se comprobó la conexión remota y la persistencia del servicio después de reiniciar Ubuntu.

Evidencias: `02-Ubuntu-Server/04Servicios-Web/` y `02-Ubuntu-Server/05Validacion-y-Persistencia/`.

## 9. Servicio web Apache

Se instaló Apache en Ubuntu Server y se publicó una página web personalizada para identificar el laboratorio.

- Servicio: `apache2`.
- Puerto TCP: 80.
- Dirección: `192.168.50.20`.
- URL: `http://srv-ubuntu01.labred.local`.

### 9.1. Configuración y validación

```bash
systemctl status apache2
systemctl is-enabled apache2
sudo apache2ctl configtest
curl -I http://localhost
```

Se configuró el nombre del servidor en:

```text
/etc/apache2/conf-available/servername.conf
```

Con el contenido:

```apache
ServerName srv-ubuntu01.labred.local
```

La configuración se habilitó y verificó mediante:

```bash
sudo a2enconf servername
sudo apache2ctl configtest
sudo systemctl reload apache2
```

Desde el cliente Windows se realizaron pruebas de resolución DNS, conectividad TCP y acceso HTTP:

```powershell
Test-NetConnection srv-ubuntu01.labred.local -Port 80
curl.exe -I http://srv-ubuntu01.labred.local
```

La respuesta HTTP documentada fue `200 OK`.

Evidencias: `02-Ubuntu-Server/04Servicios-Web/` y `02-Ubuntu-Server/05Validacion-y-Persistencia/`.

## 10. Red virtual y NAT

La infraestructura utiliza el conmutador interno `Switch-lab` y la red `192.168.50.0/24`.

El anfitrión dispone de la dirección `192.168.50.1` en su adaptador virtual y proporciona conectividad externa mediante `NAT-Laboratorio`.

Ubuntu utiliza configuración estática mediante Netplan:

- Interfaz: `eth0`.
- Dirección: `192.168.50.20/24`.
- Puerta de enlace: `192.168.50.1`.
- DNS: `192.168.50.10`.
- Dominio de búsqueda: `labred.local`.

### Verificación

En el anfitrión:

```powershell
Get-VMSwitch
Get-NetNat
Get-NetIPAddress -AddressFamily IPv4
```

En Ubuntu:

```bash
ip addr
ip route
resolvectl status
```

## 11. Pruebas de integración

Las pruebas de integración contemplaron:

1. Conectividad del cliente con Windows Server y Ubuntu.
2. Resolución DNS directa e inversa.
3. Asignación DHCP al cliente.
4. Autenticación de usuarios del dominio.
5. Aplicación de GPO y mapeo de unidades.
6. Acceso permitido y denegado a recursos SMB.
7. Conectividad TCP hacia SSH y Apache.
8. Acceso HTTP mediante nombre DNS.
9. Persistencia de servicios Linux después de reinicios.

Comandos utilizados desde el cliente:

```powershell
ipconfig /all
ping 192.168.50.10
ping 192.168.50.20
Resolve-DnsName srv-dc01.labred.local
Resolve-DnsName srv-ubuntu01.labred.local
Test-NetConnection 192.168.50.10 -Port 53
Test-NetConnection 192.168.50.20 -Port 22
Test-NetConnection 192.168.50.20 -Port 80
curl.exe -I http://srv-ubuntu01.labred.local
```

## 12. Administración de recursos virtualizados

Debido a la disponibilidad de 16 GB de RAM del equipo anfitrión, las máquinas virtuales se administraron de manera escalonada.

Las pruebas se realizaron encendiendo los servidores y el cliente según los servicios requeridos, sin mantener necesariamente toda la infraestructura activa simultáneamente.

## 13. Estado de implementación

La infraestructura principal se encuentra implementada. Se dispone de evidencias de configuración de Active Directory, DNS, DHCP, SMB, GPO, SSH y Apache, así como de pruebas de conectividad e integración.

Las actividades de mantenimiento, diagnóstico y recuperación se documentan en los procedimientos y el registro de incidencias del laboratorio.