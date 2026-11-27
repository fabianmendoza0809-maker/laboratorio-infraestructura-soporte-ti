# Registro de incidencias y soluciones

## 1. Objetivo

Documentar las incidencias presentadas durante la implementación del laboratorio, las verificaciones realizadas y los resultados obtenidos.

Este registro conserva el historial de diagnóstico y las soluciones comprobadas. También incluye validaciones preventivas y una simulación controlada de interrupción de servicio.

## 2. INC-001 — Resolución DNS del dominio desde Windows 11

**Tipo:** Incidencia de resolución de nombres.

**Descripción:**

Al intentar acceder a `http://srv-ubuntu01.labred.local`, el navegador no podía encontrar el servidor.

**Diagnóstico:**

Se ejecutaron las siguientes consultas desde Windows:

```cmd
nslookup srv-ubuntu01.labred.local
nslookup srv-ubuntu01.labred.local 192.168.50.10
```

El equipo utilizaba Google DNS (`8.8.8.8`), que no podía resolver el dominio privado `labred.local`.

La consulta dirigida al DNS interno resolvió correctamente el nombre hacia `192.168.50.20`.

**Causa identificada:**

El adaptador virtual del anfitrión no estaba utilizando el servidor DNS interno del laboratorio.

**Solución aplicada:**

Se configuró el adaptador `vEthernet (Switch-lab)` para utilizar `192.168.50.10` como servidor DNS.

Se mantuvo sin cambios la configuración DNS del adaptador Wi-Fi.

Posteriormente se limpió la caché:

```cmd
ipconfig /flushdns
```

**Resultado:**

El nombre `srv-ubuntu01.labred.local` comenzó a resolverse correctamente y la página web personalizada pudo abrirse desde el navegador.

**Estado:** Resuelta.

## 3. INC-002 — Cliente Windows sin dirección DHCP

**Tipo:** Incidencia de configuración de red.

**Descripción:**

Durante la primera prueba de asignación automática, `CLIENTE-WIN11` recibió la dirección IPv4 automática `169.254.136.73`.

Al ejecutar `ipconfig /renew`, el sistema indicó que no podía establecer contacto con el servidor DHCP.

**Diagnóstico:**

Se realizaron verificaciones en Hyper-V y Windows Server:

- Se confirmó que el cliente estaba conectado a `Switch-lab`.
- Se comprobó el estado del adaptador virtual.
- Se verificó la dirección `192.168.50.10` del servidor.
- Se confirmó que el ámbito `LAN-LABRED` estaba activo.
- Se comprobó que las reglas del firewall relacionadas con DHCP estaban habilitadas.
- Se consultaron las estadísticas del servidor, que inicialmente mostraban cero solicitudes DHCP.

**Acciones realizadas:**

Se revisaron las conexiones de red de las máquinas virtuales, el conmutador virtual, el adaptador del anfitrión, la interfaz de escucha DHCP y las reglas del firewall.

Posteriormente se volvió a ejecutar la solicitud de renovación desde el cliente.

**Resultado:**

El cliente recibió correctamente la dirección `192.168.50.100`.

Las estadísticas del servidor registraron:

| Estadística | Resultado |
|---|---:|
| Discovers | 1 |
| Offers | 1 |
| Requests | 1 |
| Acks | 1 |

La asignación automática quedó comprobada.

**Observación:**

Las verificaciones permitieron descartar varias causas, pero no se identificó una causa técnica única y definitiva para la falla inicial.

**Estado:** Resuelta.

## 4. INC-003 — Valor incorrecto en la puerta de enlace DHCP

**Tipo:** Error de configuración.

**Descripción:**

Durante la revisión de las opciones del ámbito DHCP se encontró una dirección incorrecta en la opción 003 Router:

`182.168.50.1`

La dirección correcta es `192.168.50.1`.

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

También se comprobaron las opciones:

- DNS: `192.168.50.10`.
- Dominio: `labred.local`.

**Estado:** Corregida y verificada.

## 5. INC-004 — Resolución DNS externa desde Ubuntu

**Tipo:** Incidencia de resolución de nombres.

**Descripción:**

Durante las pruebas de resolución DNS desde Ubuntu, las consultas directas al servidor DNS funcionaban, pero inicialmente la resolución mediante el servicio local presentaba problemas.

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

También se verificó que Netplan incluyera `~.` y `labred.local` en la sección de búsqueda DNS.

**Resultado:**

Se comprobó la resolución de nombres internos y externos desde Ubuntu y se ejecutó satisfactoriamente:

```bash
sudo apt update
```

Posteriormente se verificó la configuración de red y DNS después de reiniciar Ubuntu, incluyendo la resolución de nombres internos y externos.

**Estado:** Resuelta y persistencia verificada.

## 6. INC-005 — Conectividad y acceso a Internet de las máquinas virtuales

**Tipo:** Incidencia de conectividad.

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

## 8. Registro de incidencias del laboratorio

### INC-006 — Error al crear recursos compartidos SMB

**Síntoma:** `New-SmbShare` devolvía Windows System Error 1332.

**Causa:** Se utilizó el nombre `LABRED\Domain Admins`, pero el grupo administrativo del dominio tenía el nombre localizado `LABRED\Admins. del dominio`.

**Solución:** Verificar el nombre del grupo mediante Active Directory y utilizar el nombre correcto en el parámetro `-FullAccess`.

**Resultado:** Creación correcta de los tres recursos compartidos.

**Estado:** Resuelta.

### INC-007 — Error de autenticación SMB

**Síntoma:** Windows 11 devolvía el error de sistema 86 al intentar acceder con `agomez`.

**Causa:** La contraseña utilizada no era válida para la cuenta.

**Solución:** Restablecer la contraseña desde Active Directory y repetir la autenticación SMB.

**Resultado:** Autenticación correcta y acceso al recurso de Administración.

**Estado:** Resuelta.

### INC-008 — Permisos NTFS heredados

**Síntoma:** Las carpetas departamentales conservaban permisos heredados de `BUILTIN\Usuarios`.

**Causa:** Las carpetas todavía heredaban entradas de permisos de su carpeta principal.

**Solución:** Realizar una copia de seguridad de las ACL, proteger la herencia y establecer permisos explícitos para SYSTEM, administradores y el grupo departamental correspondiente.

**Resultado:** Configuración NTFS aplicada y verificada mediante `Get-Acl`.

**Estado:** Resuelta.

### INC-009 — Diagnóstico de conectividad entre cliente y servidores

**Tipo:** Validación preventiva.

**Descripción:**

Se planteó un escenario de soporte en el que un usuario informa que no puede acceder al servidor Ubuntu ni visualizar la página web interna.

**Diagnóstico:**

Desde `CLIENTE-W11-01` se realizaron pruebas mediante `Test-NetConnection`.

| Destino | Puerto | Servicio | Resultado |
|---|---:|---|---|
| 192.168.50.20 | 22 | SSH | Exitoso |
| 192.168.50.20 | 80 | Apache HTTP | Exitoso |
| 192.168.50.10 | 389 | LDAP | Exitoso |

Las tres pruebas devolvieron `TcpTestSucceeded: True`.

**Resultado:**

Se confirmó la conectividad TCP desde el cliente hacia SSH, Apache y LDAP.

Posteriormente se completaron las comprobaciones de resolución DNS y acceso HTTP mediante `srv-ubuntu01.labred.local`, obteniendo una respuesta `200 OK`.

No fue necesario modificar la infraestructura durante estas pruebas.

**Estado:** Validación completada.

### INC-010 — Configuración de resolución DNS inversa

**Tipo:** Configuración y validación DNS.

**Síntoma:**

Las consultas DNS directas funcionaban, pero al consultar las direcciones IP de los servidores mediante `nslookup`, el sistema devolvía `Non-existent domain`.

**Diagnóstico:**

Se verificó que el cliente utilizaba el DNS `192.168.50.10` y que los registros directos resolvían correctamente:

- `labred.local` → `192.168.50.10`
- `srv-dc01.labred.local` → `192.168.50.10`
- `srv-ubuntu01.labred.local` → `192.168.50.20`

La revisión de las zonas DNS permitió comprobar que no existía una zona de búsqueda inversa para la red `192.168.50.0/24`.

**Causa identificada:**

No estaban configurados los registros PTR necesarios para relacionar las direcciones IP de los servidores con sus nombres DNS.

**Solución aplicada:**

1. Se creó la zona inversa `50.168.192.in-addr.arpa`.
2. Se agregó el registro PTR para `192.168.50.10`, asociado a `srv-dc01.labred.local`.
3. Se agregó el registro PTR para `192.168.50.20`, asociado a `srv-ubuntu01.labred.local`.
4. Se realizaron nuevas consultas desde el cliente Windows 11.

**Resultado:**

Las consultas inversas permitieron resolver ambas direcciones IP hacia los nombres de sus servidores.

**Estado:** Resuelta y verificada.

**Evidencias:**

`01-Windows-Server/11-Diagnostico-Incidencias/02-DNS/`

### INC-011 — Diagnóstico y validación del servicio DHCP

**Tipo:** Validación preventiva.

**Descripción:**

Se comprobó que Windows Server proporcionaba automáticamente la configuración de red a los equipos cliente.

**Diagnóstico:**

Se verificó que el servidor DHCP estaba autorizado en Active Directory y que el ámbito `192.168.50.0/24` estaba activo.

Se comprobaron los siguientes parámetros:

- Rango: `192.168.50.100 - 192.168.50.200`.
- Puerta de enlace: `192.168.50.1`.
- Servidor DNS: `192.168.50.10`.
- Sufijo DNS: `labred.local`.

**Pruebas realizadas:**

1. Verificación de la autorización del servidor DHCP.
2. Comprobación del ámbito y sus opciones.
3. Consulta de las concesiones activas.
4. Revisión de las estadísticas DHCP.
5. Liberación y renovación de la dirección IP del cliente.
6. Comprobación de la configuración de red posterior a la renovación.
7. Pruebas de conectividad con Windows Server y Ubuntu.

**Resultado:**

`CLIENTE-W11-01` recibió nuevamente la dirección `192.168.50.101`, con máscara `255.255.255.0`, puerta de enlace `192.168.50.1` y DNS `192.168.50.10`.

El servidor registró tres concesiones activas, sin ofertas pendientes ni respuestas NAK.

No fue necesario modificar la configuración existente.

**Estado:** Validación completada. Servicio DHCP operativo.

**Evidencias:**

`01-Windows-Server/11-Diagnostico-Incidencias/03-DHCP/`

### INC-012 — Diagnóstico y validación de directivas de grupo (GPO)

**Tipo:** Validación preventiva.

**Descripción:**

Se comprobó que las directivas de grupo configuradas en Active Directory se aplicaban correctamente a los usuarios y equipos del dominio.

**Diagnóstico:**

Se verificó la existencia y vinculación de las directivas mediante `Get-GPO` y `Get-GPInheritance`.

Se revisaron las políticas de contraseñas, el mapeo de unidades de red y la seguridad de los equipos.

**Pruebas realizadas:**

1. Consulta de las GPO existentes en el dominio.
2. Verificación de los vínculos con las unidades organizativas.
3. Comprobación de la política efectiva de contraseñas.
4. Actualización de directivas mediante `gpupdate /force`.
5. Consulta de políticas aplicadas mediante `gpresult`.
6. Verificación de unidades de red para Administración, Soporte TI y Ventas.
7. Comprobación del parámetro de inactividad mediante el Registro de Windows.
8. Generación de un informe HTML con `gpresult`.

**Resultado:**

Las directivas se encontraron correctamente vinculadas y aplicadas.

Se verificó el mapeo de las unidades A:, S: y V: para sus respectivos usuarios y la configuración de inactividad de 600 segundos en el equipo cliente.

No fue necesario modificar las directivas existentes.

**Estado:** Validación completada.

**Evidencias:**

`01-Windows-Server/11-Diagnostico-Incidencias/04-GPO/`

### INC-013 — Validación de acceso SMB y permisos departamentales

**Tipo:** Validación preventiva.

**Equipos:** SRV-DC01 y CLIENTE-W11-01.

**Descripción:**

Se verificó la configuración de los recursos compartidos `Administracion`, `SoporteTI` y `Ventas`, sus permisos SMB y NTFS y el acceso de los usuarios del dominio.

**Diagnóstico:**

Se ejecutaron `Get-SmbShare`, `Get-SmbShareAccess` e `icacls` en SRV-DC01.

Se comprobaron las rutas compartidas y las entradas de permisos de cada departamento.

**Resultados:**

- Los tres recursos compartidos corresponden a sus carpetas departamentales.
- Los grupos `GG_Administracion`, `GG_SoporteTI` y `GG_Ventas` tienen permisos SMB Change y NTFS Modify sobre su recurso correspondiente.
- SYSTEM y los grupos administrativos conservan control total NTFS.
- Los usuarios acceden a sus unidades asignadas y no pueden acceder a los recursos de otros departamentos.

**Conclusión:**

La configuración de permisos SMB y NTFS funciona según el diseño de acceso departamental.

No se identificaron fallos activos ni fue necesario realizar cambios.

**Estado:** Validación completada.

**Evidencias:**

`01-Windows-Server/11-Diagnostico-Incidencias/05-Acceso-SMB/`

### INC-014 — Interrupción controlada de Apache

**Servidor:** SRV-UBUNTU01

**Cliente de validación:** CLIENTE-W11-01

**Servicio:** Apache HTTP, TCP 80

**Tipo:** Simulación de incidencia en laboratorio.

**Descripción:**

Se detuvo manualmente Apache para practicar el diagnóstico y la recuperación de un servicio web.

**Diagnóstico:**

- `systemctl is-active apache2` reportó `inactive`.
- El servicio presentó el estado `inactive (dead)`.
- Los registros confirmaron la detención a las 21:13:39 UTC del 27 de noviembre de 2026.

**Causa identificada:**

Detención manual y controlada del servicio Apache.

**Acciones de recuperación:**

1. Se validó la configuración mediante `apache2ctl configtest`, con resultado `Syntax OK`.
2. Se inició Apache mediante `systemctl start apache2`.
3. El servicio reportó `active`.
4. Los registros confirmaron el inicio a las 21:15:44 UTC.
5. Se documentó la comprobación de conectividad TCP 80 y respuesta HTTP desde CLIENTE-W11-01.

**Resultado:**

Servicio Apache recuperado y acceso web restablecido desde el cliente Windows.

Tiempo de interrupción registrado: 2 minutos y 5 segundos.

SSH permaneció disponible durante la prueba.

**Estado:** Resuelta.

**Evidencias:**

- `02-Ubuntu-Server/06-Administracion-y-Diagnostico/12-incidencia-apache-detenido.png`
- `02-Ubuntu-Server/06-Administracion-y-Diagnostico/13-recuperacion-apache.png`

## 9. Conclusión

Las incidencias registradas permitieron practicar diagnóstico de conectividad, resolución de nombres, configuración DHCP, administración de servicios y revisión de componentes de virtualización.

Las pruebas permitieron diferenciar entre errores reales de configuración, validaciones preventivas y simulaciones controladas de interrupción.

Cada problema se abordó mediante comandos de diagnóstico, comprobaciones de configuración y pruebas posteriores para verificar los resultados.

El laboratorio demuestra la importancia de documentar los síntomas, identificar las causas, aplicar soluciones y comprobar la recuperación de los servicios.
