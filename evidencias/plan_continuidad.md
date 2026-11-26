# Plan de Continuidad Operacional – ApexLogística SpA

**Alcance:** red corporativa de las sedes Santiago (casa matriz), Valparaíso y Concepción, sus enlaces WAN, el servidor SRV-Principal (DNS + Web) y el futuro controlador de dominio (Parte 2).
**Objetivo:** definir cómo se detecta, contiene y recupera cada incidente crítico, con tiempos objetivo medibles.

## Definiciones

- **RTO (*Recovery Time Objective*):** tiempo máximo aceptable entre la caída de un servicio y su restablecimiento.
- **RPO (*Recovery Point Objective*):** cantidad máxima de datos que se acepta perder, medida en tiempo desde el último respaldo válido.
- **Responsables:** *Administrador de red* (routers y enlaces), *Administrador de sistemas* (servidores, DC y web), *Mesa de ayuda* (comunicación con usuarios), *Jefatura TI* (escalamiento y decisión de activar el plan).

---

## Escenario 1: caída de un enlace (WAN con el ISP o inter-sucursal)

| Elemento | Detalle |
|---|---|
| **Impacto** | **WAN de una sucursal:** la sede pierde su salida directa a Internet, pero sigue operando. **Enlace inter-sucursal:** el tráfico entre sedes cambia de camino y aumenta la latencia. Si caen Stgo–Valpo y Valpo–Conce a la vez, una sede queda comunicada solo a través del ISP. |
| **Detección** | Interfaz `down/down` en `show ip interface brief`; pérdida del vecino OSPF en `show ip ospf neighbor` (mensaje `%OSPF-5-ADJCHG ... FULL to DOWN`); cambio del *Gateway of last resort* en `show ip route`; reclamos de usuarios; monitoreo por ping o SNMP hacia las IP de los /30. |
| **Acción inmediata** | 1. Confirmar el alcance con `show ip route` y `tracert` desde la sede afectada. 2. Verificar que la conmutación automática ocurrió: ruta flotante **AD 210** instalada y OSPF reconvergido. 3. Abrir un ticket con el proveedor del enlace. 4. Avisar a los usuarios por la mesa de ayuda si hay degradación. |
| **Recuperación** | Cuando el proveedor repara el enlace: `no shutdown` si correspondía, y verificar la adyacencia OSPF **FULL**, la vuelta de la ruta principal **AD 1** y `tracert` por el camino original. Respaldar la configuración con `copy running-config startup-config`. |
| **RTO / RPO** | **RTO ≈ 0–1 min** para el servicio, porque la conmutación es automática (ruta flotante instantánea y OSPF converge en menos de 40 s); RTO del enlace físico según el SLA del proveedor (meta 4 h). **RPO = 0**: no hay datos en juego, solo transporte. |
| **Responsable** | Administrador de red; escalamiento a Jefatura TI si la caída supera 4 h. |
| **Medidas preventivas** | Topología con **caminos redundantes**: cada sede tiene su ISP más al menos un enlace inter-sucursal. **Rutas estáticas flotantes** (AD 210) y **ruta resumida de respaldo en el ISP** (AD 250). OSPF con costos ajustados para un camino primario predecible. Contratar enlaces con SLA y, si el presupuesto lo permite, un enlace directo Santiago–Concepción o un segundo ISP. |
| **Evidencia probada** | Prueba real en Packet Tracer: `shutdown` de R-Concepcion Gi0/0. La ruta por defecto pasó a `S* [210/0] via 172.16.40.161`, el tracert a 8.8.8.8 pasó por Valparaíso y el ping dio 0 % de pérdida (ver `E6_04_prueba_falla_enlace_ISP_Concepcion.txt`). |

---

## Escenario 2: caída del Domain Controller (Active Directory / Samba – Parte 2)

| Elemento | Detalle |
|---|---|
| **Impacto** | Los usuarios no pueden iniciar sesión con cuentas del dominio en equipos donde no tengan credenciales en caché. No se aplican GPO, no se pueden cambiar contraseñas ni crear cuentas. Si el DC también resuelve DNS interno, los registros del dominio (SRV) dejan de resolverse. Los servicios ya iniciados (web, navegación) siguen funcionando. |
| **Detección** | Errores de inicio de sesión ("no hay servidores de inicio de sesión disponibles"); ping a **172.16.40.132** sin respuesta; monitoreo de servicios (Netlogon/Samba `samba-tool` / `dcdiag`); alertas de la consola de la nube (estado de la VM en AWS). |
| **Acción inmediata** | 1. Verificar si la falla es de red (ping o tracert al DC) o del servicio. 2. Si existe un **DC secundario**, confirmar que atiende las autenticaciones y que los clientes lo tienen como DNS alternativo. 3. Reiniciar el servicio o la VM si es una falla lógica. 4. Comunicar a los usuarios que no cierren sesión mientras dure el incidente. |
| **Recuperación** | Restaurar la VM desde el **último snapshot** válido o reconstruir el DC y replicar desde el secundario. Validar la replicación, los registros DNS del dominio y la aplicación de GPO (`gpupdate /force` y `gpresult` en un cliente). Documentar la causa raíz. |
| **RTO / RPO** | **RTO: 1 h** sin DC secundario (restaurar snapshot); **~0** con DC secundario. **RPO: 24 h** con snapshot diario (cambios de cuentas posteriores al snapshot deben rehacerse); **~0** con replicación entre DC. |
| **Responsable** | Administrador de sistemas; Jefatura TI decide activar la restauración. |
| **Medidas preventivas** | **DC secundario** (en otra zona de disponibilidad o sede) con replicación; **snapshots de VM** diarios con retención de 7 días; respaldo del estado del sistema; DNS secundario configurado en los clientes; monitoreo de servicios; procedimiento de restauración probado al menos semestralmente. |

---

## Escenario 3: caída del servidor web (SRV-Principal, 172.16.40.130)

| Elemento | Detalle |
|---|---|
| **Impacto** | Sin acceso a `http://www.apexlogistica.local` (inicio y documentación técnica). Como SRV-Principal también es el **DNS** de la red, si la caída es del equipo completo deja de funcionar la resolución de nombres en las tres sedes: se puede seguir accediendo por IP, pero no por nombre. |
| **Detección** | El navegador no carga la página (error de conexión o "Host Name Unresolved"); `nslookup www.apexlogistica.local` sin respuesta; ping a 172.16.40.130 fallido; monitoreo HTTP periódico de la URL. |
| **Acción inmediata** | 1. Distinguir con ping, `nslookup` y navegador si cayó solo el servicio HTTP, solo el DNS o el equipo entero. 2. Si es el servicio: reactivarlo (Services > HTTP / DNS: On). 3. Si es el equipo: levantar un **servidor alternativo** con la misma IP o actualizar el registro A hacia la IP del alternativo. Por ejemplo, en la red de servidores quedan libres .133–.142. 4. Informar a los usuarios. |
| **Recuperación** | Restaurar `index.html` y `documentacion.html` desde el **respaldo del sitio** (copia fuente en `web/` y en el repositorio de respaldo). Recrear los registros DNS (`www` y `srv`). Verificar desde un PC de cada sede con navegador y `nslookup`. |
| **RTO / RPO** | **RTO: 30 min** (servidor alternativo con el respaldo). **RPO: 24 h** con respaldo diario del sitio; **0** si el contenido está versionado al publicar cada cambio. |
| **Responsable** | Administrador de sistemas; Mesa de ayuda comunica el estado. |
| **Medidas preventivas** | **Respaldo del sitio** después de cada cambio y diario; separar DNS y Web en servidores distintos o configurar un **DNS secundario** en los clientes; servidor web de contingencia preconfigurado; snapshots si el servidor se virtualiza; monitoreo HTTP. |

---

## Resumen

| Escenario | RTO | RPO | Mecanismo principal |
|---|---|---|---|
| Caída de enlace WAN / inter-sucursal | ≈ 0–1 min (automático) | 0 | OSPF + ruta flotante AD 210 + resumen AD 250 en el ISP |
| Caída del Domain Controller | 1 h (≈ 0 con DC secundario) | 24 h (≈ 0 con replicación) | DC secundario + snapshots de VM |
| Caída del servidor web | 30 min | 24 h | Respaldo del sitio + servidor alternativo + DNS secundario |

## Mantenimiento del plan

- Probar el escenario 1 cada trimestre, con un `shutdown` controlado como el de este informe.
- Probar la restauración de snapshot (escenario 2) y del sitio (escenario 3) cada semestre.
- Actualizar el plan cuando cambie la topología, el direccionamiento o el proveedor.
