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