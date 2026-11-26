# Examen Final Integrador – Redes de Computadores (UDD)

Diseño, implementación y continuidad de la red corporativa de **ApexLogística SpA** (Santiago, Valparaíso y Concepción) en Cisco Packet Tracer.

> **Estado:** borrador. La Parte 2 (dominio Active Directory / Samba en máquinas virtuales de AWS) está pendiente.

## Contenido

| Carpeta / archivo | Descripción |
|---|---|
| `proyecto/ApexLogistica.pkt` | Proyecto de Packet Tracer con la topología y la configuración completas |
| `informe/Informe_Examen_Final.md` / `.docx` | Informe (borrador) con 32 figuras |
| `evidencias/` | Salidas de comandos (`.txt`) y capturas (`.png`) por etapa, `E1_` a `E6_` |
| `evidencias/tabla_vlsm.md` | Cálculo VLSM y tabla de direccionamiento |
| `evidencias/plan_continuidad.md` | Plan de continuidad (3 escenarios, RTO/RPO) |
| `evidencias/problemas.md` | Problemas encontrados y soluciones |
| `evidencias/checklist_rubrica.md` | Autocontrol de requisitos |
| `evidencias/guion_demo.md` | Guion de la demostración y preguntas frecuentes |
| `web/` | Fuente del sitio publicado en SRV-Principal (`index.html`, `documentacion.html`) |

## Resumen técnico

- **Direccionamiento:** VLSM sobre 172.16.40.0/22 (LAN /26 y /27, servidores /28, enlaces /30).
- **Enrutamiento:** OSPFv2 (área 0, Router-ID fijos, `passive-interface`, costo 100 en los enlaces con el ISP) más rutas estáticas por defecto, rutas flotantes de respaldo (AD 210) y un resumen /22 en el ISP (AD 250).
- **Seguridad:** SSH versión 2 (RSA de 1024 bits), `login local`, `transport input ssh`, banner MOTD, contraseñas cifradas y `exec-timeout`.
- **Servicios:** DNS (`www` y `srv.apexlogistica.local`) y HTTP en SRV-Principal.
- **Continuidad:** prueba de falla del enlace ISP–Concepción con conmutación automática a la ruta flotante.

Las contraseñas aparecen como `****` en toda la documentación.
