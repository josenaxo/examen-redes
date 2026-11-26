# Checklist de autocontrol – Examen Final Integrador (parte de red)

**Estados:** ✅ OK · ⚠️ FALTA (acción del estudiante) · ⏳ PENDIENTE-AWS (Parte 2)
**Base:** requisitos indicados por el estudiante a partir del enunciado. ⚠️ **Cruzar con el PDF oficial de la rúbrica**, que no se tuvo a la vista al armar esta lista.
**Fecha de revisión:** 25-11-2026. Proyecto: `~/pt/saves/ApexLogistica.pkt`.

## Resumen

| Estado | Cantidad |
|---|---|
| ✅ OK | 73 |
| ✅ OK parcial (falta un detalle del estudiante) | 2 |
| ⚠️ FALTA | 2 |
| ⏳ PENDIENTE-AWS | 9 |
| **Total requisitos revisados** | **86** |

**Lo que falta hacer (no AWS):**
1. ⚠️ Completar nombre y RUT en la portada del informe.
2. ⚠️ Escribir el **análisis personal** (10.2) y las **conclusiones** (10.3) del informe.
3. ⚠️ Verificar las 3 referencias marcadas [VERIFICAR] (curso NetAcad y guías de Cisco IOS).

✅ Resuelto: captura limpia de SSH Valparaíso → R-Santiago repetida desde PC-Valparaiso-2 (`E6_01`).

---

## Parte 1 – Topología, direccionamiento, seguridad y enrutamiento

### Topología
| # | Requisito | Estado | Evidencia |
|---|---|---|---|
| 1.1 | 4 routers: ISP, R-Santiago, R-Valparaiso, R-Concepcion | ✅ | E1_01 (png + txt) |
| 1.2 | Modelo y módulos con interfaces suficientes, justificados | ✅ | 2911 + HWIC-2T en Santiago y Valparaíso; justificación en E1_01.txt e informe §2.1 |
| 1.3 | Enlaces /30 ISP ↔ cada sede | ✅ | E1_01, E2_01 |
| 1.4 | Inter-sucursales Stgo–Valpo y Valpo–Conce (sin Stgo–Conce) | ✅ | E1_01.txt, E4_05 (tracert pasa por Valparaíso) |
| 1.5 | Internet simulado: Loopback0 8.8.8.8/32 en el ISP | ✅ | E2_01 (`Loopback0 8.8.8.8 up/up`) |
| 1.6 | Cada sede: switch 2960 + 2 PCs | ✅ | E1_01 |
| 1.7 | Santiago: switch de servidores con SRV-Principal y PC ADMIN | ✅ | E1_01 |
| 1.8 | Notas en el canvas (sedes, LAN, enlaces /30) | ✅ | E1_01_topologia_completa.png |
| 1.9 | Clock rate en el extremo DCE, verificado | ✅ | E2_02 (`DCE V.35, clock rate 64000` / `DTE`) |

### VLSM
| # | Requisito | Estado | Evidencia |
|---|---|---|---|
| 1.10 | VLSM sobre 172.16.40.0/22 (tabla exacta del enunciado) | ✅ | tabla_vlsm.md, E2_01 |
| 1.11 | Cálculo por subred (hosts, bits, prefijo, orden mayor a menor) | ✅ | tabla_vlsm.md §1 |
| 1.12 | Red, máscara, prefijo, rango, gateway y broadcast | ✅ | tabla_vlsm.md §2 |
| 1.13 | Observación Valparaíso /27 = 30 hosts y cómo crecer | ✅ | tabla_vlsm.md §4 |
| 1.14 | IP estáticas en PCs (primeras usables), SRV .130, ADMIN .131, DNS .130 | ✅ | E2_03 |
| 1.15 | Reserva 172.16.40.132 para el DC | ✅ | tabla_vlsm.md §3 |
| 1.16 | Espacio libre identificado (.164 – 172.16.43.255) | ✅ | tabla_vlsm.md §2 |

### Seguridad básica y SSH (4 routers)
| # | Requisito | Estado | Evidencia |
|---|---|---|---|
| 1.17 | hostname | ✅ | E2_01 / prompts |
| 1.18 | banner motd (acceso solo autorizado – ApexLogística SpA) | ✅ | E3_01 png |
| 1.19 | enable secret + service password-encryption | ✅ | E3_02.txt (`enable secret 5`) |
| 1.20 | username admin privilege 15 secret | ✅ | E3_02.txt |
| 1.21 | ip domain-name apexlogistica.local + RSA ≥ 1024 | ✅ | E3_01.txt (1024 bits en los 4) |
| 1.22 | ip ssh version 2 | ✅ | E3_02.txt (`SSH Enabled - version 2.0` ×4) |
| 1.23 | vty 0 4: login local + transport input ssh | ✅ | E3_02 png + txt |
| 1.24 | console: login local + exec-timeout | ✅ | E3_02 png + txt |
| 1.25 | Captura de verificación en un router | ✅ | E3_01, E3_02 (png) |

### Enrutamiento mixto
| # | Requisito | Estado | Evidencia |
|---|---|---|---|
| 1.26 | OSPF 1 área 0 en los 4 routers | ✅ | E4_04 (show ip protocols) |
| 1.27 | Router-ID fijos 1.1.1.1 – 4.4.4.4 | ✅ | E4_02, E4_04 |
| 1.28 | network con wildcard por cada red conectada | ✅ | E4_01, E4_04 |
| 1.29 | passive-interface en LAN y servidores | ✅ | E4_04 (Passive Interface(s)) |
| 1.30 | ip ospf cost 100 en los 3 enlaces ISP (ambos extremos) | ✅ | E4_01; costos 101/164/165 en E4_03 |
| 1.31 | Tráfico entre sedes por enlaces inter-sucursales (tracert) | ✅ | E4_05, E4_09 png |
| 1.32 | Ruta por defecto de cada sucursal hacia su ISP | ✅ | E4_03 (S\* [1/0]) |
| 1.33 | Ruta flotante AD 210 por enlace inter-sucursal | ✅ | E4_01; activa en E6_09 |
| 1.34 | ISP: resumen 172.16.40.0/22 → .146 AD 250 | ✅ | E4_02 png (`S 172.16.40.0/22 [250/0]`) |
| 1.35 | show ip ospf neighbor (todos FULL), 4 routers | ✅ | E4_02.txt; capturas E4_01, E4_03, E4_05, E4_07 |
| 1.36 | show ip route (C, L, O, S\*), 4 routers | ✅ | E4_03.txt; capturas E4_02, E4_04, E4_06, E4_08 |
| 1.37 | show ip protocols, 4 routers | ✅ | E4_04.txt |

---

## Parte 2 – Dominio en máquinas virtuales (AWS)

| # | Requisito | Estado | Evidencia |
|---|---|---|---|
| 2.1 | VMs en AWS (instancias, red, grupos de seguridad) | ⏳ PENDIENTE-AWS | – |
| 2.2 | Controlador de dominio AD / Samba (IP reservada .132) | ⏳ PENDIENTE-AWS | IP reservada en tabla_vlsm.md |
| 2.3 | Clientes unidos al dominio | ⏳ PENDIENTE-AWS | – |
| 2.4 | OU, usuarios y grupos | ⏳ PENDIENTE-AWS | – |
| 2.5 | GPO / políticas aplicadas y justificadas | ⏳ PENDIENTE-AWS | – |
| 2.6 | Informe §5 Diseño e implementación del dominio | ⏳ PENDIENTE-AWS | Estructura lista en el informe |
| 2.7 | Informe §6 Políticas aplicadas y justificación | ⏳ PENDIENTE-AWS | Estructura lista en el informe |

---

## Parte 3 – DNS y sitio web

| # | Requisito | Estado | Evidencia |
|---|---|---|---|
| 3.1 | DNS: A www.apexlogistica.local → .130 | ✅ | E5_07 png, E5_01.txt |
| 3.2 | DNS: A srv.apexlogistica.local → .130 | ✅ | E5_07 png, E5_01.txt |
| 3.3 | index.html propia (quiénes somos, sedes, servicios, enlace) | ✅ | web/index.html; E5_01, E5_03, E5_05 |
| 3.4 | documentacion.html con contenido técnico real | ✅ | web/documentacion.html; E5_02, E5_04, E5_06 |
| 3.5 | Doc: diseño de red y tabla VLSM | ✅ | documentacion.html §1 |
| 3.6 | Doc: estático + OSPF y por qué se combinan | ✅ | documentacion.html §2 |
| 3.7 | Doc: SSH y seguridad básica | ✅ | documentacion.html §3 |
| 3.8 | Doc: administración centralizada (AD/Samba, GPO) | ✅ | documentacion.html §4 |
| 3.9 | Doc: resumen del plan de continuidad | ✅ | documentacion.html §5 |
| 3.10 | Doc: referencias APA 7 | ✅ | documentacion.html §6 (solo RFC con DOI + NetAcad) |
| 3.11 | Prueba desde 1 PC por sede, por nombre, ambas páginas | ✅ | E5_01…E5_06 png, E5_02.txt |

---

## Parte 4 – Pruebas, soporte remoto y continuidad

| # | Requisito | Estado | Evidencia |
|---|---|---|---|
| 4.1 | Ping Stgo↔Valpo, Stgo↔Conce, Valpo↔Conce | ✅ | E6_01.txt (0 % pérdida) |
| 4.2 | Ping cada sede → 8.8.8.8 | ✅ | E6_01.txt |
| 4.3 | Ping cada sede → SRV-Principal | ✅ | E6_01.txt |
| 4.4 | Tracert de todos los pares anteriores | ✅ | E6_02.txt |
| 4.5 | SSH PC Valparaíso → R-Santiago | ✅ (captura limpia repetida) | E6_01 png: desde PC-Valparaiso-2, pantalla limpia con banner y `show users`. Prueba también registrada desde PC-Valparaiso-1 (E6_03.txt). La captura anterior, con sesiones anidadas, quedó en el respaldo (problema #9). |
| 4.6 | SSH PC Concepción → R-Valparaiso | ✅ | E6_02 png |
| 4.7 | SSH ADMIN → 4 routers | ✅ | E6_03…E6_06 png, E6_03.txt |
| 4.8 | (extra) Telnet bloqueado | ✅ | E6_03 png, E6_03.txt |
| 4.9 | Prueba de falla: shutdown ISP–R-Concepcion | ✅ | E6_04.txt |
| 4.10 | Reencaminamiento visible en tracert y show ip route | ✅ | E6_09, E6_10, E6_11, E6_12 |
| 4.11 | Capturas antes / después | ✅ | ANTES E6_07–08 · DURANTE E6_09–12 · DESPUÉS E6_13 |
| 4.12 | no shutdown y verificación del retorno | ✅ | E6_04.txt §5, E6_13 |
| 4.13 | plan_continuidad.md: 3 escenarios completos | ✅ | evidencias/plan_continuidad.md |
| 4.14 | Impacto, detección, acción, recuperación, RTO/RPO, responsable, prevención | ✅ | plan_continuidad.md |

---

## Parte 5 – Informe

| # | Requisito | Estado | Evidencia |
|---|---|---|---|
| 5.1 | Portada (UDD, Escuela de Ingeniería, asignatura, examen, docente, fecha) | ✅ | Informe §portada |
| 5.2 | Nombre y RUT en portada | ⚠️ FALTA | Marcado [COMPLETAR] |
| 5.3 | Índice | ✅ | Informe |
| 5.4 | Introducción | ✅ | §1 |
| 5.5 | Topología y tabla VLSM | ✅ | §2 |
| 5.6 | Enrutamiento estático y dinámico + evidencia de adyacencias | ✅ | §3, figuras 2–10 |
| 5.7 | Configuración SSH | ✅ | §4, figuras 11–12 |
| 5.8 | Diseño e implementación del dominio | ⏳ PENDIENTE-AWS | §5 (estructura) |
| 5.9 | Políticas aplicadas y justificación | ⏳ PENDIENTE-AWS | §6 (estructura) |
| 5.10 | DNS y sitio web | ✅ | §7, figuras 13–19 |
| 5.11 | Pruebas y plan de continuidad | ✅ | §8, figuras 20–32 |
| 5.12 | Problemas encontrados y soluciones | ✅ | §9 (10 casos), problemas.md |
| 5.13 | Análisis de resultados | ✅ parcial | §10.1 con datos medidos; §10.2 análisis personal ⚠️ FALTA |
| 5.14 | Conclusiones técnicas | ⚠️ FALTA | §10.3 [COMPLETAR POR EL ESTUDIANTE] |
| 5.15 | Referencias APA 7 reales | ✅ parcial | 6 RFC con DOI verificables; 3 marcadas ⚠️ [VERIFICAR] |
| 5.16 | Cada captura insertada con explicación técnica | ✅ | 32 figuras con leyenda |
| 5.17 | Versión .docx | ✅ | informe/Informe_Examen_Final.docx (32 imágenes embebidas, 31 páginas) |

---

## Archivos entregables (estado actual)

| Archivo | Ubicación |
|---|---|
| Proyecto Packet Tracer | `~/pt/saves/ApexLogistica.pkt` |
| Informe (md / docx) | `informe/Informe_Examen_Final.md`, `.docx` |
| Evidencias (50 archivos E1–E6) | `evidencias/` |
| Tabla VLSM, problemas, continuidad | `evidencias/tabla_vlsm.md`, `problemas.md`, `plan_continuidad.md` |
| Sitio web (fuente) | `web/index.html`, `web/documentacion.html` |
| Respaldo previo al renombre | `evidencias_respaldo_antes_renombre/` |
| Guion de la demo | `evidencias/guion_demo.md` |
