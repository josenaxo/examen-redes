# Guion de la demostración en vivo – parte de red

**Duración estimada:** 12–15 minutos más preguntas.
**Antes de empezar:**
- Abrir **un solo** Packet Tracer con `~/pt/saves/ApexLogistica.pkt`.
- Comprobar que todos los enlaces estén en verde.
- Tener a mano las credenciales: usuario `admin`, contraseña de usuario y enable secret (no se muestran en este documento).

**Reglas de oro durante la demo:**
- La contraseña **no se ve** al escribirla: es normal.
- Después de un `ssh` exitoso estás **dentro del router**. Antes de lanzar otro `ssh`, escribe `exit` hasta volver a `C:\>`.
- Si un ping o tracert falla la **primera vez**, repítelo: es la resolución ARP inicial.
- Si la consola de un router pide `Username:`, entra con `admin`: la sesión se cerró por `exec-timeout` (5 min), y eso es parte de la seguridad configurada.
- `show ip route` se corta en `--More--`: pulsa **espacio** para seguir.

---

## 1. Presentación de la topología (1 min)

**Muestra:** el canvas completo.

**Di:**
> "Son tres sedes y un ISP con cuatro routers 2911. Cada sede tiene su propio enlace /30 al ISP, que simula Internet con la Loopback 8.8.8.8. Entre sedes hay dos enlaces: Santiago–Valparaíso, que es serial, y Valparaíso–Concepción, que es Gigabit. No hay enlace directo Santiago–Concepción. A Santiago y Valparaíso les agregué una HWIC-2T porque necesitaban cuatro interfaces y el 2911 trae tres."

## 2. Direccionamiento VLSM (1–2 min)

**Muestra:** las notas del canvas, o `tabla_vlsm.md`.

**Di:**
> "Partí del /22 y asigné de mayor a menor: Santiago necesita 60 hosts, así que un /26 da 62. Valparaíso y Concepción van con /27, que da 30. Servidores con /28, que da 14. Los enlaces con /30. Valparaíso queda justo en 30 hosts: si crece, lo muevo a un /26 del espacio libre, que empieza en .164."

**En R-Santiago → CLI:**
```
show ip interface brief
```
> "Todas las interfaces están up/up con su IP. Serial0/0/1 queda libre como reserva."

## 3. Seguridad y SSH (2 min)

**En R-Santiago → CLI** (aparece el banner y el login):
```
admin
<contraseña>
show ip ssh
show running-config | section line
show running-config | include secret
```
> "El banner avisa que el acceso es restringido. Consola y VTY usan `login local` y las VTY solo aceptan SSH. Las contraseñas están como `secret 5`, es decir, como hash, y no en texto plano. SSH versión 2 con llave RSA de 1024 bits."

## 4. Enrutamiento OSPF + estático (3 min)

**En R-Valparaiso → CLI:**
```
show ip ospf neighbor
```
> "Valparaíso es el nodo central: tiene tres vecinos en FULL, que son el ISP, Santiago por el serial (sin DR porque es punto a punto) y Concepción."

```
show ip route
```
> "La C es una red conectada, la L mi propia IP, la O lo aprendido por OSPF y la S\* la ruta por defecto estática hacia el ISP, que es el Gateway of last resort. La ruta flotante de AD 210 no aparece porque solo entra si cae la principal."

```
show ip protocols
```
> "Proceso 1, Router-ID 3.3.3.3, las redes con su wildcard y Gi0/1 como pasiva."

**En PC-Santiago-1 → Desktop → Command Prompt:**
```
tracert 172.16.40.98
```
> "De Santiago a Concepción pasa por .158, que es Valparaíso, y por .162, que es Concepción. No pasa por el ISP, porque le puse costo 100 a los enlaces del ISP: por el camino inter-sucursal cuesta 66 y por el ISP 201."

```
tracert 8.8.8.8
```
> "Para Internet, en cambio, sale directo por su propio ISP en dos saltos, gracias a la ruta estática por defecto."

## 5. DNS y Web (1–2 min)

**En PC-Concepcion-1 → Desktop → Command Prompt:**
```
nslookup www.apexlogistica.local
```
**Luego Desktop → Web Browser:** `http://www.apexlogistica.local` y hacer clic en "Documentación Técnica".
> "El PC le pregunta al DNS 172.16.40.130, recibe el registro A y abre HTTP al mismo servidor. Desde Concepción el tráfico viaja por Valparaíso hasta la red de servidores de Santiago."

## 6. Soporte remoto por SSH (1–2 min)

**En ADMIN → Desktop → Command Prompt:**
```
ssh -l admin 172.16.40.97
<contraseña>
show users
exit
telnet 172.16.40.129
```
> "Desde la PC de gestión entro por SSH a Concepción, al otro lado de la red. `show users` muestra mi sesión en vty 0. Telnet, en cambio, el router lo cierra de inmediato, porque las VTY solo aceptan SSH."

## 7. Prueba de falla en vivo (3 min)

**Paso 1, estado normal.** En PC-Concepcion-1: `tracert 8.8.8.8` → 2 saltos.

**Paso 2, provocar la falla.** En R-Concepcion → CLI:
```
configure terminal
interface GigabitEthernet0/0
shutdown
end
show ip route
show ip ospf neighbor
```
> "Cayó el enlace con el ISP. La ruta por defecto ahora es `S* [210/0] via 172.16.40.161`: es la flotante hacia Valparaíso. OSPF perdió al ISP y solo queda el vecino 3.3.3.3."

**Paso 3.** En PC-Concepcion-1: `tracert 8.8.8.8` → 3 saltos (.97 → .161 → 8.8.8.8), y `ping 8.8.8.8` responde.
> "Concepción sigue teniendo Internet, ahora a través de Valparaíso. El ISP también aprendió por OSPF que la LAN de Concepción se alcanza por Valparaíso, así que la respuesta vuelve sin problema."

**Paso 4, restaurar** (¡no olvidar!). En R-Concepcion:
```
configure terminal
interface GigabitEthernet0/0
no shutdown
end
show ip route
```
> "Vuelve la ruta principal con AD 1 hacia el ISP y la flotante sale de la tabla."

Si la adyacencia OSPF tarda unos segundos en volver a FULL, es normal: en Ethernet espera la elección DR/BDR.

## 8. Cierre (30 s)

> "En resumen: VLSM eficiente, OSPF para la red interna, rutas estáticas con respaldo flotante para Internet, administración solo por SSH, DNS y web centralizados, y una falla de enlace que se resuelve sola. La Parte 2, el dominio en AWS, completa la administración centralizada de usuarios."

---

## 12 preguntas probables del profesor (con respuesta corta)

**1. ¿Por qué /26 para Santiago y /27 para Valparaíso y Concepción?**
Por la fórmula 2^h − 2. Santiago necesita 60 hosts: con 5 bits de host hay 30, no alcanza; con 6 hay 62, así que es un /26. Valparaíso necesita 30: con 5 bits hay exactamente 30, un /27. Concepción necesita 20: 4 bits dan 14, no alcanza; 5 bits dan 30, un /27. Siempre se elige el prefijo más pequeño que cubre la demanda, para no desperdiciar direcciones.

**2. ¿Por qué se asigna de mayor a menor?**
Para que cada subred empiece en una frontera alineada con su tamaño y no queden huecos inutilizables. Así todo el espacio libre queda contiguo al final (desde .164) y se puede usar para crecer.

**3. ¿Qué es una wildcard y cómo la calculas?**
Es la máscara invertida que usa OSPF (y las ACL) para indicar qué bits deben coincidir: un 0 obliga a coincidir y un 1 da lo mismo. Se calcula como 255.255.255.255 menos la máscara. Para un /26 (255.255.255.192) es 0.0.0.63; para un /30 es 0.0.0.3.

**4. ¿Para qué sirve `passive-interface`?**
Hace que OSPF siga anunciando la red de esa interfaz, pero sin enviar ni recibir *hellos* por ella. En una LAN de usuarios no hay routers con los que formar adyacencia, así que se ahorra tráfico y se evita que un equipo no autorizado se haga vecino e inyecte rutas.

**5. ¿Qué es la distancia administrativa? ¿Cuál es la de una ruta estática y la de OSPF?**
Es la "confiabilidad" del origen de una ruta: cuando dos fuentes ofrecen el mismo prefijo, gana la de AD menor. Conectada = 0, estática = 1, OSPF = 110. Por eso la estática por defecto (AD 1) se impone, y las rutas configuradas con 210 o 250 quedan como respaldo.

**6. ¿Qué es una ruta flotante y cómo la probaste?**
Es una ruta estática con una AD mayor que la principal (210 frente a 1). No se instala mientras la principal exista; si la principal cae, "flota" a la tabla de rutas. La probé con `shutdown` en el enlace ISP–Concepción: la ruta por defecto pasó a `S* [210/0] via 172.16.40.161` y el tracert empezó a pasar por Valparaíso.

**7. ¿Por qué combinar OSPF y rutas estáticas?**
OSPF resuelve la red interna: aprende todas las redes solo y reconverge si cae un enlace. La salida a Internet se controla mejor con estáticas, porque el proveedor no participa del OSPF de la empresa y así el camino es explícito y predecible. La ruta flotante agrega el respaldo.

**8. ¿Para qué el `ip ospf cost 100` en los enlaces del ISP?**
Sin él, OSPF podría preferir pasar por el ISP para ir de una sede a otra (Gigabit tiene costo 1). Con costo 100, el camino Santiago→Concepción por el ISP cuesta 201 y por Valparaíso 66, así que el tráfico interno usa los enlaces propios. Se ve en el tracert.

**9. ¿Cómo funciona SSH y por qué no Telnet?**
SSH negocia una sesión cifrada: el router se identifica con su llave RSA, se acuerdan claves de sesión y recién entonces se envían usuario y contraseña, cifrados. Telnet manda todo en texto plano y cualquiera que capture el tráfico vería la contraseña. Para generar la llave RSA, IOS exige hostname e `ip domain-name`. Con `transport input ssh`, Telnet queda rechazado, como se ve en la prueba.

**10. ¿Cómo se resuelve `www.apexlogistica.local`?**
El PC tiene configurado el DNS 172.16.40.130. Le consulta por el nombre, el servidor responde con el registro A 172.16.40.130, y el navegador abre una conexión TCP al puerto 80 de esa IP y pide la página (HTTP GET). `nslookup` muestra la respuesta.

**11. ¿Qué pasa si cae un enlace?**
- **Enlace ISP de una sucursal:** la estática principal deja de ser válida, entra la flotante por el enlace inter-sucursal y OSPF le avisa al ISP el nuevo camino de retorno. Es automático y sin pérdida en las pruebas.
- **Enlace inter-sucursal:** OSPF recalcula y usa el camino que queda, por ejemplo vía ISP, con más costo.

En ambos casos el RTO es de segundos y el RPO es 0.

**12. ¿Para qué sirve la ruta 172.16.40.0/22 con AD 250 en el ISP?**
Es un resumen de respaldo: cubre todo el bloque de la empresa con una sola entrada. Mientras OSPF funcione, las rutas /26, /27 y /28 son más específicas y ganan. Si OSPF fallara completo, el ISP todavía sabría devolver el tráfico hacia Santiago. Y como es AD 250, nunca le gana a OSPF.

### Preguntas extra posibles

- **¿Qué es DCE/DTE y por qué `clock rate`?** En un enlace serial, el extremo DCE genera la señal de reloj que sincroniza la transmisión. Lo verifiqué con `show controllers`: R-Santiago es DCE con 64000 y R-Valparaiso es DTE.
- **¿Por qué 2WAY y no FULL al principio?** En Ethernet, OSPF elige DR/BDR y espera unos 40 s. Entre dos DROTHER, 2WAY es normal; en un enlace con solo dos routers, finalmente queda FULL.
- **¿Qué significa `FULL/ -`?** Adyacencia completa en un enlace punto a punto (el serial), donde no se elige DR ni BDR.
- **¿Por qué el TTL del ping es 125 o 126?** Windows empieza con 128 y cada router resta 1. Un TTL de 125 significa que el paquete atravesó 3 routers (Concepción → Valparaíso → Santiago).
