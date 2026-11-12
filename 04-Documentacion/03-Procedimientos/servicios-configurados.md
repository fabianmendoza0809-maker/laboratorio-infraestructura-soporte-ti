# Procedimientos de servicios configurados

## 1. Objetivo

Documentar los principales procedimientos realizados durante la implementación del laboratorio de infraestructura TI.

Los procedimientos incluyen la configuración de Active Directory, DNS, DHCP, SSH y Apache.

## 2. Active Directory

### 2.1. Implementación

Se instaló el rol Active Directory Domain Services en Windows Server 2022.

Posteriormente, el servidor se promovió a controlador de dominio mediante la creación de un nuevo bosque.

Configuración:

- Servidor: `SRV-DC01`.
- Dominio: `labred.local`.
- NetBIOS: `LABRED`.

### 2.2. Unidades organizativas

Se creó la siguiente estructura:

```text
EmpresaLab
├── Administracion
├── SoporteTI
└── Ventas
```

Se creó el usuario de prueba `cramirez` dentro de `SoporteTI`.

### 2.3. Comprobación

```powershell
Get-ADDomain
Get-ADForest
Get-Service DNS,NTDS
```

## 3. DNS

### 3.1. Configuración

El servicio DNS se implementó en `SRV-DC01`, utilizando la dirección `192.168.50.10`.

Se verificaron las zonas:

- `labred.local`
- `_msdcs.labred.local`

Se comprobaron registros A, NS, SOA y SRV asociados al dominio.

### 3.2. Reenviadores DNS

Se configuraron los siguientes reenviadores:

```powershell
Set-DnsServerForwarder -IPAddress 8.8.8.8,8.8.4.4
```

### 3.3. Registro DNS de Ubuntu

Se creó un registro A:

- Nombre: `srv-ubuntu01`.
- Dirección IPv4: `192.168.50.20`.
- Nombre completo: `srv-ubuntu01.labred.local`.

### 3.4. Pruebas

```powershell
nslookup labred.local 127.0.0.1
nslookup google.com 127.0.0.1
```

Desde Ubuntu:

```bash
nslookup labred.local 192.168.50.10
nslookup google.com 192.168.50.10
```

Se comprobó también la resolución mediante:

```bash
resolvectl query labred.local
resolvectl query google.com
```

## 4. DHCP

### 4.1. Instalación

Se instaló el rol DHCP Server en Windows Server 2022.

Se verificó su instalación:

```powershell
Get-WindowsFeature DHCP
Get-Service DHCPServer
```

El servidor se autorizó en Active Directory.

### 4.2. Creación del ámbito

Se creó el ámbito `LAN-LABRED` con el rango:

`192.168.50.100 - 192.168.50.200`

Se configuraron las opciones:

| Opción | Valor |
|---|---|
| 003 Router | 192.168.50.1 |
| 006 DNS | 192.168.50.10 |
| 015 Dominio | labred.local |

Duración de concesión: 8 días.

### 4.3. Verificación

```powershell
Get-DhcpServerInDC
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.50.0
Get-DhcpServerv4Statistics
Get-DhcpServerv4Lease -ScopeId 192.168.50.0
```

### 4.4. Prueba desde el cliente

En `CLIENTE-WIN11`:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

Se comprobó la asignación de `192.168.50.100` y la recepción de los parámetros de red.

## 5. SSH en Ubuntu Server

### 5.1. Habilitación

Se habilitó el servicio SSH:

```bash
sudo systemctl enable --now ssh
```

Se comprobó el estado:

```bash
sudo systemctl status ssh
```

También se verificó la escucha del puerto TCP 22:

```bash
ss -tlnp | grep :22
```

### 5.2. Conexión remota

Desde Windows 11 se estableció una conexión mediante:

```powershell
ssh adminlinux@192.168.50.20
```

Se verificó el acceso a la terminal remota de Ubuntu.

## 6. Apache

### 6.1. Instalación

Se instaló Apache en Ubuntu:

```bash
sudo apt install apache2 -y
```

Se habilitó e inició el servicio:

```bash
sudo systemctl enable --now apache2
```

### 6.2. Verificación

```bash
sudo systemctl status apache2 --no-pager
curl -I http://localhost
```

Se comprobó que Apache respondiera correctamente.

### 6.3. Página personalizada

Se modificó el archivo:

`/var/www/html/index.html`

Se creó una página identificativa del laboratorio con información del servidor y del dominio.

### 6.4. Acceso desde otros equipos

Se verificó el acceso mediante:

```text
http://192.168.50.20
```

Posteriormente, se comprobó el acceso mediante:

```text
http://srv-ubuntu01.labred.local
```

## 7. NAT y conectividad

Se configuró NAT en el anfitrión Windows 11 para la red interna:

`192.168.50.0/24`

Se estableció la ruta predeterminada de los servidores hacia `192.168.50.1`.

Se verificó el acceso a direcciones externas y la resolución DNS externa.

## 8. Consideraciones operativas

Debido a la disponibilidad de memoria RAM del equipo anfitrión, las máquinas virtuales se encendieron por etapas durante algunas pruebas.

Los servidores utilizan direcciones estáticas y el cliente Windows utiliza asignación automática mediante DHCP.

## 9. Estado

Los procedimientos descritos corresponden a servicios implementados y comprobados durante el desarrollo del laboratorio.