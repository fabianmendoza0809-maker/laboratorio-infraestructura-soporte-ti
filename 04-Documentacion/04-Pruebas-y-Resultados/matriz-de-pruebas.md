# Matriz de pruebas y resultados

## 1. Objetivo

Registrar las pruebas realizadas durante la implementación del laboratorio de infraestructura y los resultados observados.

La matriz permite verificar el funcionamiento de la red virtual, Active Directory, DNS, DHCP, recursos compartidos SMB, permisos NTFS, directivas de grupo, servicios Linux y procedimientos de mantenimiento y recuperación.

## 2. Criterios de evaluación

| Estado | Significado |
|---|---|
| Correcto | La prueba se ejecutó y produjo el resultado esperado. |
| Revisada | Se realizó una comprobación diagnóstica o simulación, sin aplicar cambios. |
| Limitación | La comprobación fue satisfactoria, pero su alcance es limitado. |
| Pendiente | No se dispone de evidencia suficiente para confirmar la prueba. |

## 3. Infraestructura y conectividad

| ID | Prueba | Herramienta | Resultado obtenido | Estado |
|---|---|---|---|---|
| P01 | Conectividad Ubuntu hacia Windows Server | `ping 192.168.50.10` | 4 paquetes recibidos, 0 perdidos | Correcto |
| P02 | Conectividad cliente hacia Windows Server | `Test-NetConnection 192.168.50.10` | Conectividad confirmada desde CLIENTE-W11-01 | Correcto |
| P03 | Conectividad cliente hacia Ubuntu | `Test-NetConnection 192.168.50.20` | Conectividad confirmada desde CLIENTE-W11-01 | Correcto |
| P04 | Conectividad de Ubuntu a Internet | `ping 8.8.8.8` | Conectividad externa comprobada | Correcto |
| P05 | Actualización de índices de paquetes | `sudo apt update` | Repositorios consultados correctamente | Correcto |

## 4. Active Directory y DNS

| ID | Prueba | Herramienta | Resultado obtenido | Estado |
|---|---|---|---|---|
| P06 | Verificación del dominio | `Get-ADDomain` | Dominio `labred.local` | Correcto |
| P07 | Verificación del bosque | `Get-ADForest` | Bosque del dominio configurado | Correcto |
| P08 | Estado de servicios del dominio | `Get-Service DNS,NTDS` | Servicios en ejecución | Correcto |
| P09 | Resolución DNS interna | `nslookup labred.local 192.168.50.10` | Resolución del dominio interno | Correcto |
| P10 | Resolución DNS externa | `resolvectl query google.com` | Obtención de direcciones IP | Correcto |
| P11 | Resolución del servidor Ubuntu | `nslookup srv-ubuntu01.labred.local 192.168.50.10` | Resuelve a 192.168.50.20 | Correcto |
| P12 | Acceso web mediante DNS | Navegador web | Página personalizada visible | Correcto |
| DNS-01 | Resolución inversa Windows Server | `nslookup 192.168.50.10` | `SRV-DC01.labred.local` | Correcto |
| DNS-02 | Resolución inversa Ubuntu | `nslookup 192.168.50.20` | `srv-ubuntu01.labred.local` | Correcto |
| DNS-03 | Conectividad TCP con DNS | `Test-NetConnection 192.168.50.10 -Port 53` | `TcpTestSucceeded: True` | Correcto |

## 5. DHCP

| ID | Prueba | Herramienta | Resultado obtenido | Estado |
|---|---|---|---|---|
| P13 | Instalación del rol DHCP | `Get-WindowsFeature DHCP` | Rol instalado | Correcto |
| P14 | Estado del servicio DHCP | `Get-Service DHCPServer` | Running | Correcto |
| P15 | Autorización del servidor | `Get-DhcpServerInDC` | Servidor autorizado | Correcto |
| P16 | Estado del ámbito | `Get-DhcpServerv4Scope` | Ámbito LAN-LABRED activo | Correcto |
| P17 | Configuración de opciones DHCP | `Get-DhcpServerv4OptionValue` | Router, DNS y dominio configurados | Correcto |
| P18 | Asignación automática inicial | `ipconfig /renew` | Cliente inicial recibió 192.168.50.100 | Correcto |
| P19 | Intercambio DHCP | `Get-DhcpServerv4Lease -ScopeId 192.168.50.0` | Concesiones registradas; intercambio DHCP documentado | Correcto |
| DHCP-01 | Configuración del cliente de dominio | `ipconfig /all` | CLIENTE-W11-01 recibió 192.168.50.101/24 | Correcto |

Parámetros verificados en CLIENTE-W11-01:

- Puerta de enlace: 192.168.50.1.
- Servidor DHCP y DNS: 192.168.50.10.
- Sufijo DNS: `labred.local`.
- Duración de concesión configurada: 8 días.

## 6. Servicios Linux

| ID | Prueba | Herramienta | Resultado obtenido | Estado |
|---|---|---|---|---|
| P20 | Estado del servicio SSH | `systemctl status ssh` | Servicio activo | Correcto |
| P21 | Conexión SSH desde Windows | Cliente SSH | Acceso remoto a Ubuntu | Correcto |
| P22 | Estado del servicio Apache | `systemctl status apache2` | Servicio activo | Correcto |
| P23 | Respuesta HTTP local | `curl -I http://localhost` | Respuesta HTTP correcta | Correcto |
| P24 | Acceso web desde Windows | Navegador | Página personalizada visible | Correcto |
| LNX-01 | Conectividad SSH | `Test-NetConnection 192.168.50.20 -Port 22` | `TcpTestSucceeded: True` | Correcto |
| LNX-02 | Conectividad HTTP | `Test-NetConnection 192.168.50.20 -Port 80` | `TcpTestSucceeded: True` | Correcto |
| LNX-03 | Solicitud HTTP por DNS | `curl.exe -I http://srv-ubuntu01.labred.local` | HTTP/1.1 200 OK | Correcto |
| LNX-04 | Persistencia tras reinicio | `ip addr`, `ip route`, `resolvectl status` | IP, ruta, DNS y servicios conservados | Correcto |
| LNX-05 | Permisos de directorio Linux | `id`, `ls -ld`, pruebas de escritura | Usuario autorizado escribe; usuario ajeno denegado | Correcto |

La configuración de permisos Linux evaluada corresponde a:

- Usuario: `labsoporte1`.
- Grupo: `soporte-ti`.
- Directorio: `/srv/empresa-lab/soporte`.
- Propietario y grupo: `root:soporte-ti`.
- Permisos: `2770`.

## 7. Active Directory, SMB y NTFS

| ID | Prueba | Resultado esperado | Resultado obtenido | Estado |
|---|---|---|---|---|
| AD-01 | Organización de OU | Estructura correcta | OU EmpresaLab y sus subdivisiones | Correcto |
| AD-02 | Creación de grupos | Tres grupos departamentales | Grupos creados | Correcto |
| AD-03 | Membresía de usuarios | Cada usuario en su grupo | Membresías verificadas | Correcto |
| SMB-01 | Creación de recursos compartidos | Tres recursos disponibles | Administración, SoporteTI y Ventas | Correcto |
| SMB-02 | Acceso de agomez | Administración permitida | Acceso autorizado | Correcto |
| SMB-03 | Acceso de cramirez | SoporteTI permitido | Acceso autorizado | Correcto |
| SMB-04 | Acceso de lmartinez | Ventas permitido | Acceso autorizado | Correcto |
| SMB-05 | Acceso entre departamentos | Acceso denegado | Denegación verificada | Correcto |
| NTFS-01 | Permisos de carpetas | Acceso según grupo | Permisos NTFS aplicados | Correcto |

## 8. Integración del cliente y directivas de grupo

| ID | Prueba | Herramienta | Resultado obtenido | Estado |
|---|---|---|---|---|
| CLI-01 | Unión del cliente al dominio | Configuración de Windows | CLIENTE-W11-01 unido a labred.local | Correcto |
| CLI-02 | Inicio de sesión de dominio | Credenciales de dominio | Autenticación de usuarios del laboratorio | Correcto |
| GPO-01 | Política de contraseñas | Administración de directivas | Política de dominio configurada | Correcto |
| GPO-02 | Mapeo de unidades | `gpresult` y explorador | Unidades A:, S: y V: según departamento | Correcto |
| GPO-03 | Seguridad de equipos | `gpresult` y consulta del registro | Inactividad configurada en 600 segundos | Correcto |

## 9. Mantenimiento de Ubuntu

Fecha de las verificaciones: 27 de noviembre de 2026.

| ID | Prueba | Resultado obtenido | Estado |
|---|---|---|---|
| UB-01 | Acceso a repositorios APT | Repositorios accesibles | Correcto |
| UB-02 | Actualizaciones disponibles | 0 paquetes pendientes | Correcto |
| UB-03 | Simulación upgrade y full-upgrade | 0 paquetes por actualizar, instalar o eliminar | Correcto |
| UB-04 | Espacio en disco | 21 GB disponibles de 29 GB | Correcto |
| UB-05 | Estado SSH y Apache | Ambos servicios activos | Correcto |
| UB-06 | Validación Apache | Syntax OK | Correcto |
| UB-07 | Simulación autoremove | 6 paquetes propuestos; limpieza no ejecutada | Revisada |

## 10. Copia de seguridad y recuperación

| ID | Prueba | Resultado obtenido | Estado |
|---|---|---|---|
| UB-08 | Integridad del respaldo | Prueba gzip superada | Correcto |
| UB-09 | Contenido del respaldo | Netplan, Apache y archivos de soporte presentes | Correcto |
| UB-10 | Restauración controlada | Extracción correcta en directorio temporal | Correcto |
| UB-11 | Comparación de archivos | Netplan y prueba.txt sin diferencias | Correcto |
| UB-12 | Recuperación completa de la VM | No ejecutada | Pendiente |

Respaldo utilizado:

`/var/backups/laboratorio/backup-config-2026-11-27.tar.gz`

La recuperación se realizó en un directorio temporal, sin sobrescribir la configuración activa del servidor. No representa una restauración completa de la máquina virtual.

## 11. Prueba controlada de recuperación de Apache

| ID | Prueba | Resultado obtenido | Estado |
|---|---|---|---|
| REC-01 | Detención controlada de Apache | Servicio inactivo y habilitado | Correcto |
| REC-02 | Validación de configuración | Syntax OK | Correcto |
| REC-03 | Inicio del servicio | Apache activo nuevamente | Correcto |
| REC-04 | Verificación de registros | Inicio y detención registrados en journalctl | Correcto |

La prueba permitió comprobar el procedimiento de recuperación del servicio sin afectar la configuración de red ni el acceso SSH.

## 12. Evidencias

Las evidencias se encuentran organizadas en las carpetas del repositorio:

- `01-Windows-Server/`: instalación, Active Directory, usuarios, grupos, SMB, NTFS y GPO.
- `02-Ubuntu-Server/`: configuración de red, administración Linux, servicios web, diagnóstico, mantenimiento y respaldos.
- `03-DNS-DHCP/`: configuración DNS, registros, ámbito DHCP y pruebas.

Las evidencias de administración Linux incluyen:

- `02-Ubuntu-Server/03-Administracion-Linux/05-usuarios-y-grupos.png`
- `02-UbuntuServer/03-Administracion-Linux/06-permisos-directorio-compartido.png`
- `02-Ubuntu-Server/03-Administracion-Linux/07-prueba-acceso-grupo.png`

Las evidencias de mantenimiento y recuperación se encuentran en:

- `02-Ubuntu-Server/07-Mantenimiento-y-Actualizaciones/`
- `02-Ubuntu-Server/08-Copias-de-Seguridad/`

## 13. Consideraciones

Las pruebas de integración se realizaron encendiendo las máquinas virtuales por etapas debido a las limitaciones de memoria RAM del anfitrión.

Los resultados corresponden a las verificaciones ejecutadas durante el desarrollo del laboratorio. Las simulaciones de actualización y limpieza no implican que los paquetes hayan sido instalados o eliminados.

La prueba de respaldo demuestra recuperación de archivos seleccionados, pero no garantiza recuperación ante la pérdida completa de la máquina virtual.

## 14. Resultado general

**Resultado general: pruebas de los servicios principales completadas satisfactoriamente.**

Se comprobó el funcionamiento de la infraestructura virtual, el dominio, la resolución DNS, la asignación DHCP, los recursos compartidos, las directivas de grupo y los servicios SSH y Apache.

También se verificaron procedimientos de diagnóstico, mantenimiento y recuperación controlada. Las pruebas pendientes y las limitaciones de alcance se mantienen identificadas para futuras ampliaciones del laboratorio.
