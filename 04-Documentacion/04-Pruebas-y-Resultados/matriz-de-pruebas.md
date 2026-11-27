# Matriz de pruebas y resultados

## 1. Objetivo

Registrar las pruebas realizadas durante la implementación del laboratorio y los resultados observados.

## 2. Pruebas de infraestructura y conectividad

| ID | Prueba | Herramienta | Resultado | Estado |
|---|---|---|---|---|
| P01 | Conectividad Ubuntu hacia Windows Server | `ping 192.168.50.10` | 4 paquetes recibidos, 0 perdidos | Correcto |
| P02 | Conectividad cliente hacia Windows Server | `ping 192.168.50.10` | Respuestas recibidas | Correcto |
| P03 | Conectividad cliente hacia Ubuntu | `ping 192.168.50.20` | Respuestas recibidas | Correcto |
| P04 | Conectividad de Ubuntu a Internet | `ping 8.8.8.8` | Respuestas recibidas | Correcto |
| P05 | Actualización de índices de paquetes Ubuntu | `sudo apt update` | Repositorios consultados correctamente | Correcto |

## 3. Pruebas de Active Directory y DNS

| ID | Prueba | Herramienta | Resultado | Estado |
|---|---|---|---|---|
| P06 | Verificación del dominio | `Get-ADDomain` | Dominio `labred.local` | Correcto |
| P07 | Verificación del bosque | `Get-ADForest` | Bosque del dominio configurado | Correcto |
| P08 | Estado de servicios del dominio | `Get-Service DNS,NTDS` | Servicios verificados en ejecución | Correcto |
| P09 | Resolución DNS interna | `nslookup labred.local 192.168.50.10` | Resuelve a `192.168.50.10` | Correcto |
| P10 | Resolución DNS externa | `resolvectl query google.com` | Obtención de direcciones IP | Correcto |
| P11 | Resolución del servidor Ubuntu | `nslookup srv-ubuntu01.labred.local 192.168.50.10` | Resuelve a `192.168.50.20` | Correcto |
| P12 | Acceso mediante nombre DNS desde Windows 11 | Navegador web | Página personalizada visible | Correcto |

## 4. Pruebas DHCP

| ID | Prueba | Herramienta | Resultado | Estado |
|---|---|---|---|---|
| P13 | Instalación del rol DHCP | `Get-WindowsFeature DHCP` | Rol instalado | Correcto |
| P14 | Estado del servicio DHCP | `Get-Service DHCPServer` | Running | Correcto |
| P15 | Autorización del servidor | `Get-DhcpServerInDC` | Servidor autorizado | Correcto |
| P16 | Estado del ámbito | `Get-DhcpServerv4Scope` | Ámbito LAN-LABRED activo | Correcto |
| P17 | Configuración de opciones DHCP | `Get-DhcpServerv4OptionValue` | Router, DNS y dominio configurados | Correcto |
| P18 | Asignación automática de IP | `ipconfig /renew` | Cliente recibió `192.168.50.100` | Correcto |
| P19 | Intercambio DHCP | `Get-DhcpServerv4Lease -ScopeId 192.168.50.0` | Discover, Offer, Request y ACK registrados | Correcto |

## 5. Pruebas de servicios Linux

| ID | Prueba | Herramienta | Resultado | Estado |
|---|---|---|---|---|
| P20 | Estado del servicio SSH | `systemctl status ssh` | Servicio activo | Correcto |
| P21 | Conexión SSH desde Windows | Cliente SSH | Acceso a terminal de Ubuntu | Correcto |
| P22 | Estado del servicio Apache | `systemctl status apache2` | Servicio activo | Correcto |
| P23 | Respuesta HTTP local | `curl -I http://localhost` | Respuesta HTTP correcta | Correcto |
| P24 | Acceso web desde Windows | Navegador | Página personalizada visible | Correcto |

## 6. Resultados generales

Las pruebas permitieron comprobar:

- Comunicación entre máquinas virtuales.
- Funcionamiento del dominio y sus servicios asociados.
- Resolución DNS interna y externa.
- Asignación automática de direcciones IP mediante DHCP.
- Administración remota de Ubuntu mediante SSH.
- Publicación y acceso a una página web mediante Apache.
- Acceso al servicio web por dirección IP y nombre DNS.

## 7. Consideraciones

Las pruebas de integración que requieren varios equipos se realizaron encendiendo las máquinas virtuales por etapas, debido a las limitaciones de memoria RAM del anfitrión.

Los resultados registrados corresponden a las verificaciones ejecutadas durante el desarrollo del laboratorio.

## 8. Estado

**Resultado general: pruebas de los servicios principales completadas satisfactoriamente.**

## Pruebas de Active Directory, SMB y NTFS

| ID | Prueba | Resultado esperado | Resultado obtenido |
|---|---|---|---|
| AD-01 | Organización de OU | Estructura correcta | Correcto |
| AD-02 | Creación de grupos | Tres grupos departamentales | Correcto |
| AD-03 | Membresía de usuarios | Cada usuario en su grupo | Correcto |
| SMB-01 | Creación de recursos compartidos | Tres recursos disponibles | Correcto |
| SMB-02 | Acceso de agomez | Administración permitida | Correcto |
| SMB-03 | Acceso de cramirez | SoporteTI permitido | Correcto |
| SMB-04 | Acceso de lmartinez | Ventas permitido | Correcto |
| NTFS-01 | agomez accede a SoporteTI y Ventas | Acceso denegado | Correcto |
| NTFS-02 | cramirez accede a Administración y Ventas | Acceso denegado | Correcto |
| NTFS-03 | lmartinez accede a Administración y SoporteTI | Acceso denegado | Correcto |
| CLI-01 | Acceso desde Windows 11 | Autenticación SMB correcta | Correcto |

### Prueba: persistencia de Ubuntu Server después de reinicio

**Objetivo:** comprobar que la configuración de red y los servicios de Ubuntu Server permanecen operativos después de reiniciar la máquina virtual.

**Equipo:** SRV-UBUNTU01.

**Procedimiento:** se verificó el estado y la habilitación de los servicios SSH y Apache, se reinició el servidor y se restableció la conexión SSH desde el equipo Windows.

**Resultados:**
- La interfaz eth0 conservó la dirección IP 192.168.50.20/24.
- La ruta predeterminada continuó apuntando a 192.168.50.1.
- El servidor DNS permaneció configurado como 192.168.50.10.
- SSH y Apache se encontraron activos después del reinicio.
- Apache respondió con HTTP 200 OK mediante el nombre srv-ubuntu01.labred.local.

**Resultado final:** prueba satisfactoria. Se verificó la persistencia de la configuración de red y de los servicios principales tras el reinicio.

**Evidencias:** `02-Ubuntu-Server/05-Validacion-y-Persistencia/`

### Prueba: administración de usuarios, grupos y permisos en Ubuntu Server

**Objetivo:** comprobar que un usuario autorizado puede acceder al directorio compartido de soporte TI y que un usuario ajeno al grupo no puede escribir en él.

**Equipo:** SRV-UBUNTU01.

**Configuración evaluada:**
- Usuario de laboratorio: labsoporte1.
- Grupo de trabajo: soporte-ti.
- Directorio compartido: /srv/empresa-lab/soporte.
- Propietario del directorio: root.
- Grupo propietario: soporte-ti.
- Permisos del directorio: 2770.

**Procedimiento:**

1. Se comprobó la existencia del usuario y su pertenencia al grupo mediante `id labsoporte1`.
2. Se verificaron el propietario, el grupo y los permisos del directorio mediante `ls -ld /srv/empresa-lab/soporte`.
3. Se intentó crear un archivo de prueba utilizando la identidad del usuario autorizado.
4. Se intentó crear un archivo utilizando la cuenta de servicio nobody, ajena al grupo de soporte.
5. Se verificó que el usuario autorizado pudiera crear el archivo y que el usuario no autorizado recibiera una denegación de acceso.

**Resultados esperados:**
- El usuario labsoporte1 pertenece al grupo soporte-ti.
- El directorio pertenece a root:soporte-ti y tiene permisos 2770.
- El archivo creado por labsoporte1 hereda el grupo soporte-ti.
- La cuenta nobody no puede crear archivos dentro del directorio.

**Resultado obtenido:** [Completar con los resultados reales de las pruebas].

**Conclusión:** [Indicar si los permisos permitieron el acceso autorizado y bloquearon el acceso no autorizado].

**Evidencias:** `02-Ubuntu-Server/03-Administracion-Linux/05-usuarios-y-grupos.png`, `06-permisos-directorio-compartido.png` y `07-prueba-acceso-grupo.png`.

### Prueba: administración y diagnóstico de Ubuntu Server

**Objetivo:** verificar el estado de los servicios SSH y Apache, los puertos en escucha, los registros del sistema y la disponibilidad de actualizaciones del servidor Ubuntu.

**Equipos:** SRV-UBUNTU01 y CLIENTE-W11-01.

**Procedimiento:**

1. Se verificó el estado de SSH y Apache mediante systemctl.
2. Se comprobó la ausencia de unidades fallidas mediante systemctl --failed.
3. Se revisaron los puertos en escucha mediante ss.
4. Se consultaron los registros de SSH y Apache mediante journalctl.
5. Se corrigió el aviso de Apache AH00558 mediante la configuración global de ServerName.
6. Se actualizaron los índices de paquetes y se consultaron las actualizaciones disponibles.
7. Se verificó desde el cliente Windows la resolución DNS y la conectividad con los puertos 22 y 80.

**Resultados obtenidos:**
- SSH y Apache se encontraban activos y habilitados.
- No se identificaron unidades fallidas.
- Los puertos 22 y 80 estaban en escucha.
- Los registros mostraron conexiones SSH aceptadas y reinicios controlados de los servicios.
- El aviso de Apache correspondía a eventos anteriores a la corrección de ServerName.
- Se identificaron 87 paquetes actualizables, incluidos paquetes de seguridad.
- No se identificaron paquetes retenidos.

**Resultado de las pruebas desde el cliente:** [Registrar los resultados reales de DNS, SSH y HTTP].

**Conclusión:** los servicios principales del servidor se encuentran operativos. La revisión de actualizaciones permite planificar una ventana de mantenimiento sin confundir la disponibilidad de paquetes con su instalación efectiva.

**Evidencias:** `02-Ubuntu-Server/06-Administracion-y-Diagnostico/`

## Prueba de mantenimiento y actualizaciones de Ubuntu

**Objetivo:** comprobar la disponibilidad de actualizaciones y evaluar su instalación antes de modificar el servidor.

**Equipo:** SRV-UBUNTU01.

**Procedimiento:**
1. Actualizar el índice de paquetes mediante `sudo apt update`.
2. Consultar las actualizaciones disponibles con `apt list --upgradable`.
3. Simular la instalación mediante `sudo apt-get -s upgrade`.
4. Crear un punto de control de la máquina virtual en Hyper-V.

**Resultado esperado:** identificar los paquetes pendientes y disponer de un punto de recuperación antes de aplicar actualizaciones.

**Resultado obtenido:** [Completar según la simulación y la creación del punto de control].

**Evidencias:**
- `02-Ubuntu-Server/07-Mantenimiento-y-Actualizaciones/01-simulacion-actualizaciones.png`
- `02-Ubuntu-Server/07-Mantenimiento-y-Actualizaciones/02-punto-control-hyperv.png`

## Prueba de copia de seguridad y recuperación de Ubuntu

**Objetivo:** verificar la creación, integridad y recuperación de una copia de seguridad de configuraciones y datos del laboratorio.

**Equipo:** SRV-UBUNTU01.

**Procedimiento:**
1. Crear el directorio `/var/backups/laboratorio`.
2. Generar una copia comprimida de Netplan, Apache y `/srv/empresa-lab`.
3. Comprobar el contenido del archivo.
4. Extraer el respaldo en un directorio temporal.
5. Comparar los archivos recuperados con los originales y verificar la integridad del archivo comprimido.

**Resultado obtenido:** se creó el respaldo `backup-config-2026-11-27.tar.gz` de 35 KB. Se verificó la recuperación de los archivos y la integridad del respaldo sin sobrescribir la configuración activa del servidor.

**Limitación:** el respaldo se encuentra dentro de la misma máquina virtual. Para una recuperación ante pérdida de la VM, se requiere una copia externa.

**Evidencias:**
- `02-Ubuntu-Server/08-Copias-de-Seguridad/01-creacion-backup.png`
- `02-Ubuntu-Server/08-Copias-de-Seguridad/02-contenido-backup.png`
- `02-Ubuntu-Server/08-Copias-de-Seguridad/03-prueba-recuperacion.png`
- `02-Ubuntu-Server/08-Copias-de-Seguridad/04-integridad-backup.png`


### Conclusión

Las pruebas confirmaron que los tres usuarios pueden autenticarse desde Windows 11 y acceder a los recursos compartidos correspondientes a sus departamentos. También se verificó la denegación de acceso a los recursos de otros departamentos.

Durante estas pruebas, el equipo Windows 11 todavía pertenecía a WORKGROUP. Su incorporación al dominio queda como una actividad posterior.