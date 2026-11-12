# Registro de incidencias y soluciones

## 1. Objetivo

Documentar las incidencias presentadas durante la implementación del laboratorio, las verificaciones realizadas y los resultados obtenidos.

Este registro permite conservar el historial de diagnóstico y las soluciones comprobadas.

## 2. Incidencia 01: resolución DNS del dominio desde Windows 11

**Descripción:**

Al intentar acceder a la página web mediante:

`http://srv-ubuntu01.labred.local`

el navegador no podía encontrar el servidor.

**Diagnóstico:**

Se ejecutaron consultas DNS desde Windows 11:

```cmd
nslookup srv-ubuntu01.labred.local
```

El resultado mostró que el equipo estaba utilizando Google DNS (`8.8.8.8`), que no podía resolver el dominio privado `labred.local`.

Se realizó una segunda consulta directamente al servidor DNS interno:

```cmd
nslookup srv-ubuntu01.labred.local 192.168.50.10
```

Esta consulta resolvió correctamente el nombre hacia `192.168.50.20`.

**Solución aplicada:**

Se configuró el adaptador virtual `vEthernet (Switch-lab)` del anfitrión para utilizar `192.168.50.10` como servidor DNS.

Se mantuvo sin cambios la configuración DNS del adaptador Wi-Fi.

Posteriormente se limpió la caché DNS:

```cmd
ipconfig /flushdns
```

**Resultado:**

El nombre `srv-ubuntu01.labred.local` comenzó a resolverse correctamente y la página web personalizada pudo abrirse desde el navegador.

**Estado:** Resuelta.

## 3. Incidencia 02: cliente Windows sin dirección DHCP

**Descripción:**

Durante la primera prueba de asignación automática, `CLIENTE-WIN11` recibió una dirección IPv4 automática:

`169.254.136.73`

Al ejecutar:

```cmd
ipconfig /renew
```

apareció un mensaje indicando que no se podía establecer contacto con el servidor DHCP.

**Diagnóstico:**

Se realizaron verificaciones en Hyper-V y Windows Server:

- Se confirmó que el cliente estaba conectado a `Switch-lab`.
- Se comprobó que el adaptador virtual del cliente estaba en estado correcto.
- Se verificó que el adaptador de Windows Server utilizaba `192.168.50.10`.
- Se confirmó que el ámbito `LAN-LABRED` estaba activo.
- Se comprobó que las reglas del firewall relacionadas con DHCP estaban habilitadas.
- Se consultaron las estadísticas del servidor, que inicialmente mostraban cero solicitudes DHCP.

**Acciones realizadas:**

Se revisaron las conexiones de red de las máquinas virtuales, el conmutador virtual, el adaptador del anfitrión, la interfaz de escucha DHCP y las reglas del firewall.

Posteriormente se volvió a ejecutar la solicitud de renovación desde el cliente.

**Resultado:**

El cliente recibió correctamente la dirección:

`192.168.50.100`

Las estadísticas del servidor registraron:

| Estadística | Resultado |
|---|---:|
| Discovers | 1 |
| Offers | 1 |
| Requests | 1 |
| Acks | 1 |

La asignación automática quedó comprobada.

**Observación:** las verificaciones permitieron descartar varias causas, pero no se identificó una causa técnica única y definitiva para la falla inicial.

**Estado:** Resuelta.

## 4. Incidencia 03: valor incorrecto en la opción de puerta de enlace DHCP

**Descripción:**

Durante la revisión de las opciones del ámbito DHCP se encontró una dirección incorrecta en la opción 003 Router:

`182.168.50.1`

La dirección correcta de la puerta de enlace del laboratorio es:

`192.168.50.1`

**Solución aplicada:**

Se corrigió la opción mediante PowerShell:

```powershell
Set-DhcpServerv4OptionValue -ScopeId 192.168.50.0 -Router 192.168.50.1
```

Se verificó nuevamente:

```powershell
Get-DhcpServerv4OptionValue -ScopeId 192.168.50.0
```

**Resultado:**

La opción 003 quedó configurada con `192.168.50.1`.

También se comprobaron las opciones DNS y dominio:

- DNS: `192.168.50.10`.
- Dominio: `labred.local`.

**Estado:** Corregida y verificada.

## 5. Incidencia 04: resolución DNS externa desde Ubuntu

**Descripción:**

Durante las pruebas de resolución DNS desde Ubuntu, las consultas directas al servidor DNS funcionaban, pero inicialmente la resolución mediante el servicio local de resolución del sistema presentaba problemas.

**Diagnóstico:**

Se realizaron consultas:

```bash
resolvectl status eth0
resolvectl query google.com
resolvectl query labred.local
```

Se comprobó que el servidor DNS configurado era `192.168.50.10`, pero el dominio de enrutamiento DNS de la interfaz no permitía inicialmente resolver nombres externos mediante el comportamiento esperado.

**Solución aplicada:**

Se realizó una prueba temporal:

```bash
sudo resolvectl domain eth0 '~.'
```

Después se comprobó:

```bash
resolvectl query google.com
resolvectl query labred.local
```

Ambas consultas funcionaron correctamente.

También se verificó que el archivo de configuración de Netplan incluyera `~.` y `labred.local` en la sección de búsqueda DNS.

**Resultado:**

Se comprobó la resolución de nombres internos y externos desde Ubuntu y se ejecutó satisfactoriamente:

```bash
sudo apt update
```

**Estado:** Resolución comprobada. La persistencia del comportamiento tras un reinicio debe verificarse posteriormente.

## 6. Incidencia 05: conectividad y acceso a Internet de las máquinas virtuales

**Descripción:**

Durante la implementación se presentaron dificultades iniciales para que las máquinas virtuales accedieran a redes externas.

**Diagnóstico y acciones:**

Se verificaron las rutas de red, la comunicación con el adaptador virtual del anfitrión y la configuración NAT.

Se configuró el NAT `NAT-Laboratorio` con el prefijo:

`192.168.50.0/24`

Se estableció `192.168.50.1` como siguiente salto para los servidores.

También se configuraron reenviadores DNS en Windows Server.

**Resultado:**

Ubuntu pudo comunicarse con Internet y consultar los repositorios mediante `apt update`.

Windows Server pudo comunicarse con direcciones externas y resolver nombres externos mediante su servicio DNS.

**Estado:** Conectividad externa comprobada.

## 7. Consideraciones de recursos

El equipo anfitrión dispone de 16 GB de memoria RAM.

Debido al consumo de memoria de las máquinas virtuales, no siempre fue posible mantener todos los equipos encendidos simultáneamente.

Se optó por encender las máquinas virtuales por etapas, según las pruebas que se necesitaban realizar.

Esta estrategia permitió completar las pruebas de integración sin requerir que toda la infraestructura permaneciera encendida permanentemente.

## 8. Conclusión

Las incidencias registradas permitieron practicar diagnóstico de conectividad, resolución DNS, configuración DHCP, administración de servicios y revisión de componentes de virtualización.

Cada problema se abordó mediante comandos de diagnóstico, comprobaciones de configuración y pruebas posteriores para verificar los resultados.