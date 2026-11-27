# Mantenimiento de Ubuntu Server

## 1. Objetivo

Documentar las actividades de mantenimiento preventivo realizadas en `SRV-UBUNTU01`, incluyendo la revisión de recursos, actualización de paquetes, diagnóstico de servicios, comprobación de respaldos y validación posterior al mantenimiento.

El procedimiento busca mantener la disponibilidad del servidor y reducir el riesgo de interrupciones en los servicios SSH y Apache.

## 2. Entorno de trabajo

| Parámetro | Configuración |
|---|---|
| Servidor | SRV-UBUNTU01 |
| Sistema operativo | Ubuntu Server 26.04.1 LTS |
| Dirección IPv4 | 192.168.50.20/24 |
| Puerta de enlace | 192.168.50.1 |
| DNS interno | 192.168.50.10 |
| Interfaz de red | eth0 |
| Usuario administrativo | adminlinux |
| Servicios principales | SSH y Apache |
| Plataforma de virtualización | Hyper-V |

## 3. Comprobaciones previas

Antes de iniciar el mantenimiento, se revisaron el estado del sistema, el almacenamiento disponible, la configuración de red y los servicios activos.

### 3.1. Sistema y recursos

```bash
hostname
lsb_release -a
uname -r
uptime
free -h
df -h
```

### 3.2. Conectividad

```bash
ip addr
ip route
resolvectl status
resolvectl query srv-dc01.labred.local
resolvectl query google.com
```

Estas comprobaciones permiten verificar que el servidor conserva su dirección IP, la ruta predeterminada y la resolución de nombres mediante el DNS interno.

### 3.3. Servicios

```bash
systemctl status ssh --no-pager
systemctl status apache2 --no-pager
systemctl is-enabled ssh
systemctl is-enabled apache2
sudo apache2ctl configtest
```

## 4. Revisión de actualizaciones

### 4.1. Actualización de los índices

Se ejecutó:

```bash
sudo apt update
```

El comando consultó los repositorios de Ubuntu correspondientes a `resolute`, incluidos los canales de actualizaciones, seguridad y backports.

Posteriormente se revisaron los paquetes disponibles:

```bash
apt list --upgradable
apt-mark showhold
```

### 4.2. Resultado de la revisión inicial

En la revisión inicial se identificaron 87 paquetes actualizables. Entre ellos se encontraban componentes relacionados con:

- Seguridad y autenticación: `libpam0g`, `libpam-modules`, `libgcrypt20`, `sudo` y `sudo-rs`.
- Red: `netplan.io`, `netplan-generator`, `libnetplan1` y `python3-netplan`.
- Sistema: `systemd-resolved`, `libsystemd0`, `libudev1`, `rsyslog` y `snapd`.
- Herramientas y bibliotecas: `git`, `python3-apt`, `libsqlite3-0` y `libxml2`.

Esta revisión correspondió a un estado anterior del servidor. La cantidad de paquetes disponibles puede variar después de actualizar los índices o aplicar mantenimiento.

La consulta `apt-mark showhold` permite identificar paquetes retenidos. No se documentó una lista de paquetes retenidos en la salida inicial.

## 5. Simulación de actualizaciones

Antes de aplicar cambios, se revisa el impacto de las actualizaciones mediante simulaciones:

```bash
apt -s upgrade
apt -s full-upgrade
apt -s autoremove
```

Las simulaciones permiten identificar paquetes que serían actualizados, instalados o eliminados sin realizar modificaciones en el sistema.

La operación `full-upgrade` puede proponer cambios de dependencias más amplios que `upgrade`. La operación `autoremove` puede identificar paquetes que APT considera innecesarios.

No se debe asumir que todos los paquetes propuestos para eliminación son prescindibles, especialmente cuando están relacionados con el arranque o la administración del sistema.

## 6. Resultado del mantenimiento del 27 de noviembre de 2026

Durante la revisión posterior se documentaron los siguientes resultados:

| Comprobación | Resultado |
|---|---|
| Índices de APT | Repositorios consultados correctamente |
| Paquetes actualizables | No se identificaron paquetes pendientes |
| Simulación de upgrade | 0 actualizaciones, 0 instalaciones y 0 eliminaciones |
| Simulación de full-upgrade | 0 actualizaciones, 0 instalaciones y 0 eliminaciones |
| Kernel en ejecución | 7.0.0-44-generic |
| Espacio del sistema | Aproximadamente 29 GB totales, 6,2 GB utilizados y 21 GB disponibles |
| Utilización del disco | 23 % |
| SSH | Activo |
| Apache | Activo |
| Configuración Apache | Syntax OK |

La revisión inicial había mostrado el kernel `7.0.0-30`. En la comprobación posterior se encontraba en ejecución `7.0.0-44-generic`.

### 6.1. Revisión de paquetes candidatos a eliminación

La simulación de `autoremove` propuso seis paquetes:

- `grub-pc-bin`
- `libftdi1-2`
- `libjson-glib-1.0-0`
- `libflashrom1`
- `libjcat1`
- `libjson-glib-1.0-common`

No se documentó su eliminación efectiva. Se decidió conservarlos, especialmente por la presencia de componentes relacionados con el arranque.

La simulación de una operación no equivale a su ejecución.

## 7. Validación posterior al mantenimiento

Las comprobaciones posteriores permiten verificar que los servicios principales continúan funcionando.

```bash
uname -r
df -h /
systemctl is-active ssh
systemctl is-active apache2
sudo apache2ctl configtest
sudo ss -tlnp
curl -I http://localhost
```

También se comprobó la conectividad desde Windows:

```powershell
Test-NetConnection srv-ubuntu01.labred.local -Port 22
Test-NetConnection srv-ubuntu01.labred.local -Port 80
curl.exe -I http://srv-ubuntu01.labred.local
```

La respuesta HTTP documentada fue `200 OK`.

## 8. Revisión de registros del sistema

Para el diagnóstico de servicios se utilizaron los siguientes comandos:

```bash
sudo journalctl -u ssh -n 30 --no-pager
sudo journalctl -u apache2 -n 30 --no-pager
sudo journalctl -p err -b --no-pager
```

En los registros iniciales de Apache se identificó la advertencia `AH00558`, relacionada con la ausencia de una directiva global `ServerName`.

La advertencia no impidió el inicio del servicio. Posteriormente se configuró:

```text
/etc/apache2/conf-available/servername.conf
```

Con el contenido:

```apache
ServerName srv-ubuntu01.labred.local
```

Se habilitó la configuración y se comprobó la sintaxis antes de recargar Apache:

```bash
sudo a2enconf servername
sudo apache2ctl configtest
sudo systemctl reload apache2
```

En el registro de errores también se observó una conexión SSH reiniciada por el cliente `192.168.50.101`. Este evento aislado no demuestra una caída del servicio SSH.

## 9. Respaldo de configuración

Como parte del mantenimiento se realizó un respaldo de archivos seleccionados del servidor.

Archivo documentado:

```text
/var/backups/laboratorio/backup-config-2026-11-27.tar.gz
```

El archivo tenía un tamaño aproximado de 35 KB y contenía configuraciones de red, archivos de Apache y archivos del directorio de práctica de soporte.

Se comprobó la integridad del archivo comprimido:

```bash
BACKUP=/var/backups/laboratorio/backup-config-2026-11-27.tar.gz

sudo gzip -t "$BACKUP"
sudo tar -tzf "$BACKUP"
```

El comando `gzip -t` finalizó sin errores y el listado permitió identificar los archivos incluidos.

Este respaldo es parcial: no constituye una copia completa del sistema operativo ni una imagen recuperable de toda la máquina virtual.

## 10. Prueba de restauración controlada

Para comprobar que el respaldo podía extraerse sin sobrescribir la configuración activa, se utilizó un directorio temporal independiente.

La ruta de restauración documentada fue:

```text
/var/tmp/restauracion-lab.ZN7nmY
```

La extracción se realizó con:

```bash
BACKUP=/var/backups/laboratorio/backup-config-2026-11-27.tar.gz
LAB=/var/tmp/restauracion-lab.ZN7nmY

sudo tar --no-same-owner --no-same-permissions \
  -xzf "$BACKUP" -C "$LAB"
```

La extracción finalizó sin errores.

Se identificaron el archivo Netplan y los archivos de prueba `prueba.txt` y `prueba-acceso.txt`. Los archivos de prueba tenían un tamaño de cero bytes.

La comparación del archivo Netplan restaurado con el original no produjo diferencias:

```bash
sudo diff -u /etc/netplan/00-installer-config.yaml \
  "$LAB/etc/netplan/00-installer-config.yaml"
```

La ausencia de salida indica que ambos archivos eran idénticos en esa comparación.

La prueba no sobrescribió configuraciones activas ni requirió reiniciar los servicios. La extracción correcta demuestra la recuperación de los archivos seleccionados, pero no equivale a una restauración completa del servidor.

## 11. Incidencia controlada de Apache

El 27 de noviembre se documentó una prueba de detención y recuperación del servicio Apache.

| Evento | Resultado |
|---|---|
| Detención de Apache | 21:13:39 UTC |
| Estado observado | inactive (dead), enabled |
| Inicio de Apache | 21:15:44 UTC |
| Duración de la interrupción | 2 minutos y 5 segundos |
| Validación de sintaxis | Syntax OK |
| Recuperación | Servicio activo |

La detención fue intencional y no una falla espontánea del servidor.

Se verificó el estado del servicio mediante:

```bash
systemctl status apache2 --no-pager
sudo apache2ctl configtest
sudo systemctl start apache2
systemctl is-active apache2
```

SSH permaneció disponible durante la prueba. El detalle de la incidencia se encuentra en `05-Incidencias-y-Soluciones/registro-de-incidencias.md`.

## 12. Evidencias

Las evidencias del mantenimiento se encuentran en:

```text
02-Ubuntu-Server/
├── 05-Validacion-y-Persistencia/
├── 06-Administracion-y-Diagnostico/
├── 07-Mantenimiento-y-Actualizaciones/
└── 08copias-de-seguridad/
```

Las capturas documentan la revisión de actualizaciones, las simulaciones de APT, los registros de servicios, la creación del respaldo y la extracción controlada.

## 13. Conclusión

El mantenimiento permitió comprobar el estado de los paquetes, el almacenamiento y los servicios de Ubuntu Server.

La revisión posterior no mostró actualizaciones pendientes y las simulaciones de APT no propusieron cambios adicionales de paquetes.

La validación del respaldo confirmó que los archivos seleccionados podían extraerse en un directorio independiente, sin afectar la configuración activa.

Finalmente, la prueba controlada de Apache permitió documentar el diagnóstico, la recuperación del servicio y la importancia de comprobar su disponibilidad después de una interrupción.
