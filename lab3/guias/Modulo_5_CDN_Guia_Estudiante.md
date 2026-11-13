# Módulo 5. Redes de Distribución de Contenidos (CDN)

**Curso:** Enlaces Internacionales de Internet  
**Duración de exposición:** 30 minutos  
**Destinatarios:** estudiantes con conocimientos básicos de redes IP  
**Enfoque:** arquitectura, interconexión y economía de los ISP de Bolivia  
**Modalidad:** lectura teórica y demostraciones en vivo. Los ejercicios se desarrollarán en la guía de laboratorio independiente.

## Objetivos de aprendizaje

Al finalizar el módulo, el estudiante podrá explicar qué es una CDN y cómo se relaciona con los enlaces internacionales; distinguir un servidor de origen de una caché o servidor de borde; describir el papel de DNS, anycast, BGP y HTTP en la selección y entrega de contenido; reconocer distintos modelos de despliegue de CDN en ISP; y utilizar herramientas web y de línea de comandos para encontrar indicios de uso de CDN sin confundirlos con pruebas concluyentes de ubicación física.

## 6.1. ¿Por qué existen las CDN?

Internet nació como una interconexión de redes, pero la distribución masiva de páginas, software, imágenes y video plantea un problema práctico: si millones de usuarios solicitan los mismos objetos a un único servidor lejano, el servidor y las rutas de transporte pueden convertirse en cuellos de botella. Una **Content Delivery Network** (CDN) es una infraestructura distribuida que acerca copias de ciertos contenidos o funciones de procesamiento a las redes desde las cuales se solicitan. En vez de atender todas las solicitudes desde el servidor de origen, una CDN puede responder desde uno de sus puntos de presencia (POP).

La mejora no proviene únicamente de reducir kilómetros. También depende de capacidad disponible, acuerdos de interconexión, procesamiento TLS, optimización de conexiones, caché y políticas de enrutamiento. Una CDN puede reducir el tiempo de respuesta y el volumen de datos que debe atravesar enlaces internacionales, pero no garantiza por sí sola ninguna de las dos cosas.

**Ejemplo:** un usuario en La Paz solicita una imagen publicada por un sitio cuyo servidor de origen está en Norteamérica. Si la imagen está almacenada en un nodo accesible por interconexión local o regional, la respuesta puede evitar un trayecto completo al origen. Si no está almacenada, el nodo deberá recuperarla y, según la política de caché, conservar una copia para solicitudes posteriores.

## 6.2. Arquitectura de una CDN

### Origen, POP, edge y caché

- **Servidor de origen (origin):** sistema que conserva la versión principal del contenido o genera respuestas dinámicas.
- **POP (Point of Presence):** ubicación donde el proveedor despliega equipos y conectividad; puede albergar servidores de borde, routers y otros componentes.
- **Edge server:** servidor que atiende solicitudes próximas a los usuarios en términos de conectividad, no necesariamente de distancia geográfica.
- **Caché:** almacenamiento temporal de objetos que pueden reutilizarse; puede existir en múltiples niveles (edge, regional, shield).
- **Cache hit:** el objeto solicitado se entrega desde una caché válida.
- **Cache miss:** la caché no dispone de una respuesta utilizable y debe buscarla en un nivel superior o en el origen.
- **TTL de caché:** tiempo durante el cual una respuesta puede considerarse reutilizable según las políticas aplicables. No debe confundirse con el TTL de un registro DNS.

### Flujo simplificado

```text
Usuario en Bolivia
       |
       | 1. Resolución DNS / establecimiento de conexión
       v
Nodo de borde de la CDN (POP)
       |
       +---- HIT ----> Entrega desde caché ----> Usuario
       |
       +---- MISS ---> Caché regional / servidor de origen
                             |
                             v
                     Respuesta y posible almacenamiento
                             |
                             v
                           Usuario
```

La arquitectura real puede incluir balanceadores, autenticación, cifrado TLS, reglas de caché, origen múltiple y protección contra ataques. **No todo el contenido es cacheable**: páginas personalizadas, respuestas privadas y transmisiones en tiempo real pueden requerir tratamientos distintos. Una CDN también puede operar como proxy inverso sin servir necesariamente un objeto desde caché.

### HTTP y política de caché

Encabezados como `Cache-Control`, `Age`, `ETag` y `Vary` influyen en cómo se reutilizan las respuestas. Algunos proveedores añaden encabezados propios como `CF-Cache-Status`, `X-Cache` o `Server-Timing`; su presencia y significado dependen del servicio y de su configuración. Un encabezado que indique `HIT` puede ser evidencia de una respuesta desde caché, pero **no revela por sí solo la ciudad ni el país del servidor**.

## 6.3. ¿Cómo llega el usuario al nodo de la CDN?

### DNS y selección de destino

Una CDN puede responder a consultas DNS con distintas direcciones IP según el resolutor, la ubicación aproximada, la carga o políticas operativas. También puede emplear CNAME para asociar un dominio del cliente con un nombre administrado por la CDN. **La dirección IP obtenida por DNS no identifica necesariamente el servidor de origen**: puede ser la de un proxy o un punto de entrada.

La ubicación del resolutor recursivo no siempre coincide con la del usuario. Algunas implementaciones utilizan **EDNS Client Subnet (ECS)** para aportar una indicación parcial de la red cliente, con implicaciones de privacidad y precisión. Por ello, dos resolutores pueden devolver respuestas diferentes y dos usuarios que obtienen la misma IP pueden terminar en ubicaciones distintas.

### Anycast y BGP

En **anycast**, múltiples ubicaciones anuncian una misma dirección o prefijo IP. BGP dirige el tráfico hacia una de las ubicaciones alcanzables según las políticas y condiciones de enrutamiento. El destino elegido no es necesariamente el más cercano geográficamente ni el de menor latencia. Esto conecta directamente el presente módulo con el estudio de ASN y BGP.

### Transporte y protocolos

Las CDN entregan contenido mediante HTTP/HTTPS sobre TCP o QUIC (HTTP/3). La terminación TLS, la reutilización de conexiones, la compresión y los protocolos de transporte pueden mejorar el rendimiento percibido. Un cambio de rendimiento no prueba, por sí solo, que el contenido se haya movido a una caché local.

## 6.4. Principales modelos y proveedores

No todas las redes de distribución funcionan con el mismo modelo de negocio:

| Modelo | Ejemplos | Rasgo principal |
|---|---|---|
| CDN comercial de propósito general | Cloudflare, Akamai, Amazon CloudFront | Distribución para sitios y aplicaciones de múltiples clientes |
| Red propia de plataforma de contenido | Netflix Open Connect, Google Global Cache | Optimización de contenido de una plataforma concreta |
| Infraestructura de contenido integrada con ISP | Equipos de caché o interconexión directa acordados con proveedores | Puede disminuir dependencia de transporte hacia redes externas |

**Netflix Open Connect** puede desplegar appliances (OCA) en redes participantes o entregar tráfico mediante interconexiones. **Google Global Cache (GGC)** permite almacenar determinados contenidos de Google en infraestructura próxima a los usuarios de ISP participantes. La existencia de estos programas no implica que estén presentes en todos los ISP de Bolivia. La presencia, ubicación y capacidad deben comprobarse caso por caso.

En las CDN comerciales, el proveedor puede tener POP propios, infraestructura arrendada o acuerdos de colocación e interconexión. Un mismo proveedor puede utilizar estrategias diferentes en distintos países.

## 6.5. CDN, IXP y conectividad internacional de Bolivia

Un ISP boliviano puede recibir contenido mediante distintas rutas técnicas y comerciales:

1. **Tránsito IP:** el ISP obtiene el contenido a través de un proveedor que le da acceso a Internet global.
2. **Peering directo:** el ISP intercambia tráfico con la red de contenido en una interconexión bilateral.
3. **Peering en un IXP:** el intercambio se establece en una infraestructura compartida, si ambas redes están presentes y acuerdan interconectarse.
4. **Caché dentro del ISP:** determinados objetos se entregan desde servidores ubicados en su propia red o infraestructura próxima.
5. **Conexión regional:** el contenido se obtiene en un POP de un país vecino, que puede reducir la dependencia de rutas más largas, aunque sigue siendo tráfico externo al país.

### Tres situaciones para comparar

```text
A) Sin caché accesible localmente:
Usuario -> ISP Bolivia -> transporte/tránsito internacional -> CDN/origen externo

B) Con peering regional:
Usuario -> ISP Bolivia -> enlace regional -> POP CDN en país vecino

C) Con caché dentro del ISP:
Usuario -> red ISP Bolivia -> caché local
                             |
                             +-- si MISS --> red externa / origen
```

**Advertencia:** una conexión de baja latencia no demuestra que el servidor esté en Bolivia. Una IP registrada en Bolivia tampoco garantiza que el equipo esté físicamente allí. La geolocalización comercial de IP, las rutas BGP y los traceroute son indicios complementarios, no certificados de ubicación física.

### Relación con los módulos anteriores

- **Fibra óptica:** transporta el tráfico entre el usuario, los POP y el origen.
- **DNS:** ayuda a localizar el punto de entrada del servicio.
- **ASN:** identifica redes participantes y sus políticas administrativas.
- **BGP:** determina alcanzabilidad y contribuye a seleccionar rutas hacia los POP.
- **CDN:** decide dónde y cómo se responde a las solicitudes de contenido.

## 6.6. Demostraciones web en vivo

Estas demostraciones son para acompañar la exposición. No constituyen actividades evaluables.

### Demo A. Identificar indicios de CDN en un dominio

**Sitio:** [CDN Finder — CDN Planet](https://www.cdnplanet.com/tools/cdnfinder/)

Introducir un dominio público y observar si la herramienta identifica un proveedor CDN. Explicar que la detección se basa en señales técnicas y puede producir resultados incompletos o no detectar configuraciones personalizadas.

### Demo B. Analizar recursos y proveedores de un sitio

**Sitio:** [Cloudflare Radar URL Scanner](https://radar.cloudflare.com/scan)

Examinar un dominio público y revisar las solicitudes, dominios, IP y ASN que muestre el informe. Distinguir entre el dominio principal y los dominios que alojan imágenes, scripts o video. Un sitio puede utilizar más de una CDN.

### Demo C. Investigar redes de contenido y su interconexión

**Sitios:** [PeeringDB](https://www.peeringdb.com/) y [BGP.Tools](https://bgp.tools/)

Buscar un proveedor de CDN o de contenido y revisar sus redes, ubicaciones e interconexiones declaradas. PeeringDB es una base de datos mantenida por sus participantes: la presencia declarada en un IXP no garantiza que dos operadores mantengan una sesión de peering activa. BGP.Tools muestra información observada e inferida; no sustituye contratos privados.

### Demo D. Medir el rendimiento desde distintas ubicaciones

**Sitio:** [WebPageTest](https://www.webpagetest.org/)

Seleccionar, cuando estén disponibles, ubicaciones de prueba distintas para una misma URL. Comparar TTFB, tiempos de descarga y solicitudes. Las diferencias pueden deberse a caché, red, carga, TLS y otros factores; evitar atribuirlas automáticamente a distancia física.

### Demo E. Observar contexto regional de tráfico y red

**Sitio:** [Cloudflare Radar](https://radar.cloudflare.com/)

Explorar vistas disponibles por país y métricas relacionadas con tráfico, protocolos y conectividad. Las mediciones de un proveedor reflejan su propia visibilidad y metodología: no representan necesariamente la totalidad del tráfico de Internet de Bolivia.

## 6.7. Demostraciones con `dig`, `curl` y `traceroute`

Las siguientes órdenes están pensadas para una demostración en Debian/Ubuntu. Los resultados cambian con el tiempo, la ubicación y la configuración de red.

### Instalación de utilidades

```bash
sudo apt update
sudo apt install dnsutils curl traceroute
```

### Demo F. Comparar respuestas DNS

```bash
dig www.cloudflare.com A +short
dig @1.1.1.1 www.cloudflare.com A +short
dig @8.8.8.8 www.cloudflare.com A +short
dig www.cloudflare.com AAAA +short
```

Si aparecen IP distintas, explicar que pueden intervenir políticas DNS, balanceo y caché. Si son iguales, ello no implica necesariamente que los paquetes lleguen al mismo centro de datos: anycast puede usar la misma IP en múltiples ubicaciones.

### Demo G. Examinar encabezados HTTP

```bash
curl -I https://www.cloudflare.com/
curl -sS -D - -o /dev/null https://www.cloudflare.com/
```

Buscar, cuando existan, encabezados de caché y otros indicadores de la CDN. Algunos servidores no permiten `HEAD` o generan respuestas diferentes para `HEAD` y `GET`; por eso se incluyen ambas variantes. No se debe asumir que siempre aparecerá `CF-Cache-Status` o que una respuesta `HIT` implique caché en Bolivia.

### Demo H. Observar una ruta IP

```bash
traceroute -n www.cloudflare.com
```

Comentar saltos visibles, variaciones de RTT y límites del método: MPLS, filtrado ICMP, rutas asimétricas y balanceo pueden ocultar tramos. Un traceroute no muestra directamente la ruta física de fibra ni los contratos de tránsito.

### Demo I. Comparar la IP observada con un ASN

Obtener una IP con `dig`, luego consultar esa dirección en [BGP.Tools](https://bgp.tools/) o [RIPEstat](https://stat.ripe.net/). Identificar el prefijo y ASN de origen anunciados públicamente. **El ASN que origina el prefijo no es necesariamente el propietario del servidor de origen del sitio web.**

## 6.8. Impacto económico en un ISP

Supongamos un ISP que transporta en un intervalo determinado **10 Gbps de tráfico agregado hacia usuarios**. Si **6 Gbps** se sirven desde cachés dentro de su propia red y los **4 Gbps** restantes provienen de redes externas, en ese intervalo la demanda externa es menor que si todo el tráfico tuviera que recuperarse fuera del ISP.

| Concepto | Valor hipotético |
|---|---:|
| Tráfico total entregado a usuarios | 10 Gbps |
| Tráfico servido por cachés internas | 6 Gbps |
| Tráfico recibido desde redes externas | 4 Gbps |
| Proporción local en ese intervalo | 60 % |

Este ejemplo **no significa** que el ISP pueda reducir automáticamente su factura de tránsito en 60 %. La facturación puede depender de capacidad comprometida, percentil 95, contratos mínimos, mezcla de tráfico, picos, redundancia y costos de transporte. Las cachés también consumen energía, espacio, puertos y capacidad interna.

### Capacidad frente a tráfico facturable

- **Capacidad del enlace:** velocidad nominal disponible, por ejemplo 10 Gbps.
- **Tráfico instantáneo:** utilización medida en un momento dado.
- **Percentil 95:** método de facturación que ordena muestras de utilización y descarta aproximadamente el 5 % más alto, según el contrato.
- **Ahorro potencial:** depende de cómo la caché cambie la curva de utilización, especialmente los períodos que determinan la facturación.

Además, un cache miss puede provocar tráfico de llenado hacia el origen. No todo byte servido desde una CDN equivale a un byte internacional evitado: la comparación requiere medir dónde se aloja la caché y cómo llega el contenido a ella.

## 6.9. Límites de lo que podemos observar públicamente

Para interpretar correctamente las demostraciones conviene distinguir entre evidencia directa e inferencia:

| Observación | Qué permite afirmar | Qué no demuestra |
|---|---|---|
| DNS devuelve una IP | El resolutor obtuvo esa respuesta | La ubicación física del servidor |
| BGP anuncia un prefijo desde un ASN | Ese ASN aparece como origen en la vista consultada | Quién posee el contenido original |
| `CF-Cache-Status: HIT` u otro indicador equivalente | Según el proveedor, hubo una respuesta desde caché | Que la caché esté en Bolivia |
| Traceroute de pocos saltos | Se observaron pocos routers que respondieron | La ruta óptica exacta o la distancia física |
| Registro en PeeringDB | Una red declara presencia o política de interconexión | Una sesión bilateral activa y su contrato |
| Baja latencia | La comunicación tuvo RTT bajo en la medición | Que el contenido esté alojado localmente |

## 6.10. Síntesis del módulo

Una CDN es una capa de distribución de contenidos y servicios que se apoya en la infraestructura física y lógica de Internet. El usuario consulta un nombre mediante DNS, alcanza direcciones IP anunciadas por redes identificadas con ASN, atraviesa rutas seleccionadas mediante BGP y recibe una respuesta desde un POP, una caché o un origen. Para los ISP bolivianos, el valor de una CDN depende de **dónde está interconectada, qué contenido entrega, cuánta demanda absorbe y cómo se factura la conectividad externa**.

La principal conclusión es que **proximidad geográfica, proximidad de red y ahorro económico son conceptos relacionados, pero diferentes**.

## Referencias y sitios de consulta

- [Cloudflare Learning Center — What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)
- [Netflix Open Connect](https://openconnect.netflix.com/)
- [Google Global Cache](https://peering.google.com/#/infrastructure)
- [Amazon CloudFront Documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [MDN — HTTP Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111)
- [RFC 4271 — Border Gateway Protocol 4](https://www.rfc-editor.org/rfc/rfc4271)
- [PeeringDB](https://www.peeringdb.com/)
- [BGP.Tools](https://bgp.tools/)
- [RIPEstat](https://stat.ripe.net/)
- [Cloudflare Radar](https://radar.cloudflare.com/)
- [CDN Planet CDN Finder](https://www.cdnplanet.com/tools/cdnfinder/)
- [WebPageTest](https://www.webpagetest.org/)

---

**Nota curricular:** este módulo corresponde a la exposición teórica con demostraciones del docente. La guía de laboratorio de una hora se elaborará por separado y reunirá las actividades prácticas de todos los módulos.
