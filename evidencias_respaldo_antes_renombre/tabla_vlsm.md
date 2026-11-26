# Tabla VLSM – ApexLogística SpA

**Bloque asignado:** 172.16.40.0/22 (172.16.40.0 – 172.16.43.255, 1024 direcciones)
**Método:** VLSM (*Variable Length Subnet Masking*): se asignan las subredes **de mayor a menor** cantidad de hosts. Así cada subred empieza en una frontera alineada con su tamaño y el espacio libre queda contiguo al final del bloque.

## 1. Cálculo de cada subred

Fórmula: hosts utilizables = 2^h − 2, donde *h* = bits de host. Se restan 2 direcciones porque la primera identifica la red y la última es el broadcast. Para cada subred se elige el menor *h* que cubra los hosts pedidos y el prefijo resulta /(32 − h).

| Orden | Subred | Hosts pedidos | Bits de host (h) | Hosts disponibles (2^h − 2) | Prefijo | Por qué ese prefijo |
|---|---|---|---|---|---|---|
| 1 | LAN Santiago | 60 | 6 | 62 | /26 | Con h = 5 solo hay 30 hosts (< 60); con h = 6 hay 62 ≥ 60. Sobran 2. |
| 2 | LAN Valparaíso | 30 | 5 | 30 | /27 | Con h = 4 hay 14 (< 30); con h = 5 hay exactamente 30. Sobran 0 (ver §4). |
| 3 | LAN Concepción | 20 | 5 | 30 | /27 | Con h = 4 hay 14 (< 20); con h = 5 hay 30 ≥ 20. Sobran 10. |
| 4 | Servidores / Mgmt | 10 | 4 | 14 | /28 | Con h = 3 hay 6 (< 10); con h = 4 hay 14 ≥ 10. Sobran 4. |
| 5–9 | Enlaces punto a punto (×5) | 2 | 2 | 2 | /30 | Un enlace entre 2 routers necesita exactamente 2 IP; /30 es el mínimo sin desperdicio. |

## 2. Tabla de direccionamiento

| Subred | Red | Máscara | Prefijo | Rango usable | Gateway / Extremos | Broadcast |
|---|---|---|---|---|---|---|
| LAN Santiago | 172.16.40.0 | 255.255.255.192 | /26 | .1 – .62 | GW .1 (R-Santiago Gi0/1) | 172.16.40.63 |
| LAN Valparaíso | 172.16.40.64 | 255.255.255.224 | /27 | .65 – .94 | GW .65 (R-Valparaiso Gi0/1) | 172.16.40.95 |
| LAN Concepción | 172.16.40.96 | 255.255.255.224 | /27 | .97 – .126 | GW .97 (R-Concepcion Gi0/1) | 172.16.40.127 |
| Servidores / Mgmt | 172.16.40.128 | 255.255.255.240 | /28 | .129 – .142 | GW .129 (R-Santiago Gi0/2) | 172.16.40.143 |
| ISP – Santiago | 172.16.40.144 | 255.255.255.252 | /30 | .145 – .146 | ISP Gi0/0 .145 / R-Santiago Gi0/0 .146 | 172.16.40.147 |
| ISP – Valparaíso | 172.16.40.148 | 255.255.255.252 | /30 | .149 – .150 | ISP Gi0/1 .149 / R-Valparaiso Gi0/0 .150 | 172.16.40.151 |
| ISP – Concepción | 172.16.40.152 | 255.255.255.252 | /30 | .153 – .154 | ISP Gi0/2 .153 / R-Concepcion Gi0/0 .154 | 172.16.40.155 |
| Santiago – Valparaíso | 172.16.40.156 | 255.255.255.252 | /30 | .157 – .158 | R-Santiago Se0/0/0 .157 (DCE) / R-Valparaiso Se0/0/0 .158 | 172.16.40.159 |
| Valparaíso – Concepción | 172.16.40.160 | 255.255.255.252 | /30 | .161 – .162 | R-Valparaiso Gi0/2 .161 / R-Concepcion Gi0/2 .162 | 172.16.40.163 |
| **Libre (crecimiento)** | 172.16.40.164 | – | – | 172.16.40.164 – 172.16.43.255 | – | – |

Internet simulado: Loopback0 del ISP = **8.8.8.8/32**. Está fuera del bloque de la empresa.

**Uso del bloque:** 164 de 1024 direcciones (≈ 16 %). Quedan 860 libres para crecimiento.

## 3. Asignación de hosts

| Equipo | Interfaz | IP | Máscara | Gateway | DNS |
|---|---|---|---|---|---|
| PC-Santiago-1 | Fa0 | 172.16.40.2 | 255.255.255.192 | 172.16.40.1 | 172.16.40.130 |
| PC-Santiago-2 | Fa0 | 172.16.40.3 | 255.255.255.192 | 172.16.40.1 | 172.16.40.130 |
| PC-Valparaiso-1 | Fa0 | 172.16.40.66 | 255.255.255.224 | 172.16.40.65 | 172.16.40.130 |
| PC-Valparaiso-2 | Fa0 | 172.16.40.67 | 255.255.255.224 | 172.16.40.65 | 172.16.40.130 |
| PC-Concepcion-1 | Fa0 | 172.16.40.98 | 255.255.255.224 | 172.16.40.97 | 172.16.40.130 |
| PC-Concepcion-2 | Fa0 | 172.16.40.99 | 255.255.255.224 | 172.16.40.97 | 172.16.40.130 |
| SRV-Principal (DNS + Web) | Fa0 | 172.16.40.130 | 255.255.255.240 | 172.16.40.129 | 172.16.40.130 |
| ADMIN (management) | Fa0 | 172.16.40.131 | 255.255.255.240 | 172.16.40.129 | 172.16.40.130 |
| **Domain Controller (Parte 2 – reservado)** | – | **172.16.40.132** | 255.255.255.240 | 172.16.40.129 | – |

Se dejan libres en la red de servidores las direcciones 172.16.40.133 – .142 (10 IP) para futuros servidores, por ejemplo un DC secundario o un servidor de respaldo.

## 4. Observación: Valparaíso queda justo en 30 hosts

El /27 de Valparaíso entrega **exactamente** 30 direcciones utilizables, una de las cuales es el gateway (.65). Si la sede necesita **un solo host más** (31 en total), el /27 no alcanza. Opciones de crecimiento:

1. **Migrar a un /26** (62 hosts). No se puede extender en el lugar porque 172.16.40.96/27 (Concepción) es la subred contigua. Habría que tomar un /26 alineado del espacio libre, por ejemplo **172.16.40.192/26** (.193 – .254, GW .193), y renumerar la LAN de Valparaíso.
2. **Agregar una segunda subred** (por ejemplo 172.16.41.0/27) en otra interfaz o subinterfaz/VLAN de R-Valparaiso y anunciarla por OSPF.

Gracias al orden de mayor a menor, todo el espacio libre (172.16.40.164 en adelante) es contiguo, así que cualquiera de estas opciones cabe sin reorganizar las demás sedes.
