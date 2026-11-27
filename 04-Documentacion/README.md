# Documentación técnica

Esta carpeta contiene la documentación complementaria del proyecto **Laboratorio de Infraestructura y Soporte TI**.

Su propósito es describir la arquitectura implementada, el direccionamiento de red, los procedimientos de configuración, las pruebas realizadas y las incidencias encontradas durante el desarrollo del laboratorio.

## 1. Estructura de la documentación

### 01. Arquitectura

Archivo: `01-Arquitectura/TOPOLOGIA-DE-RED.md`

Describe los componentes del laboratorio, las máquinas virtuales, sus funciones y la conexión entre los equipos.

### 02. Direccionamiento IP

Archivo: `02-Direccionamiento-IP/plan-de-direccionamiento.md`

Contiene el esquema IPv4, las direcciones estáticas, el rango DHCP, el DNS y la puerta de enlace.

### 03. Procedimientos

Archivo: `03-Procedimientos/servicios-configurados.md`

Documenta las principales actividades de configuración de Active Directory, DNS, DHCP, SSH y Apache.

### 04. Pruebas y resultados

Archivo: `04-Pruebas-y-Resultados/matriz-de-pruebas.md`

Registra las verificaciones ejecutadas, los resultados observados y el estado de cada prueba.

### 05. Incidencias y soluciones

Archivo: `05-Incidencias-y-Soluciones/registro-de-incidencias.md`

Describe los problemas encontrados durante la implementación, las acciones de diagnóstico y los resultados obtenidos.

## 2. Administración de usuarios, grupos y recursos compartidos

### 2.1. Organización de Active Directory

Se organizó el dominio `labred.local` mediante la unidad organizativa principal `EmpresaLab`, que contiene las siguientes OU:

- `Administracion`
- `SoporteTI`
- `Ventas`
- `Grupos`
- `Equipos`

La estructura permite separar las cuentas de usuario, los grupos de seguridad y los equipos según su función dentro del entorno empresarial simulado.

### 2.2. Usuarios y grupos de seguridad

Se crearon tres grupos globales de seguridad:

- `GG_Administracion`
- `GG_SoporteTI`
- `GG_Ventas`

Se crearon tres usuarios y se asignaron a sus respectivos grupos:

| Usuario | Departamento | Grupo |
|---|---|---|
| `agomez` | Administración | `GG_Administracion` |
| `cramirez` | Soporte TI | `GG_SoporteTI` |
| `lmartinez` | Ventas | `GG_Ventas` |

Se verificaron las cuentas, su ubicación en las unidades organizativas y la pertenencia a los grupos correspondientes.

### 2.3. Recursos compartidos SMB

Se crearon tres recursos compartidos en el servidor:

| Recurso | Ruta local |
|---|---|
| `Administracion` | `C:\EmpresaLab\Administracion` |
| `SoporteTI` | `C:\EmpresaLab\SoporteTI` |
| `Ventas` | `C:\EmpresaLab\Ventas` |

Se concedió permiso de modificación al grupo correspondiente a cada departamento y control total a los administradores del dominio.

### 2.4. Permisos NTFS

Se protegió la herencia de permisos en las tres carpetas departamentales y se establecieron permisos explícitos:

- `SYSTEM`: FullControl.
- `BUILTIN\Administradores`: FullControl.
- `LABRED\Admins. del dominio`: FullControl.
- Grupo departamental correspondiente: Modify.

La configuración se verificó mediante la inspección de las ACL de las carpetas.

### 2.5. Acceso desde Windows 11

Se verificó el acceso mediante SMB utilizando credenciales de dominio desde un equipo Windows 11 que inicialmente pertenece a `WORKGROUP`.

El equipo cliente no necesita estar unido al dominio para realizar estas pruebas de autenticación SMB.

Se comprobó que:

- `agomez` puede acceder al recurso `Administracion`.
- `cramirez` puede acceder al recurso `SoporteTI`.
- `lmartinez` puede acceder al recurso `Ventas`.
- Los usuarios reciben acceso denegado al intentar ingresar a los recursos de otros departamentos.

Los resultados permiten comprobar la aplicación de los permisos de acceso configurados en el servidor.

## 3. Evidencias del proyecto

Las capturas de pantalla se encuentran organizadas en las carpetas correspondientes del repositorio.

Entre las evidencias disponibles se incluyen:

- Creación y configuración de máquinas virtuales.
- Instalación y configuración de Windows Server.
- Configuración de dirección IP estática.
- Creación y validación del dominio.
- Servicios DNS y Active Directory.
- Estructura de unidades organizativas.
- Usuarios y grupos de seguridad.
- Recursos compartidos SMB.
- Permisos SMB y NTFS.
- Pruebas de acceso autorizado y denegación entre departamentos.


## 4. Criterios de documentación

- Se documentan configuraciones y resultados comprobados.
- Las direcciones IP corresponden a la red privada del laboratorio.
- Las capturas de pantalla se conservan en las carpetas de evidencias correspondientes.
- No se incluyen contraseñas, claves privadas ni información confidencial.
- Las actividades pendientes se identifican como tales y no se presentan como implementadas.