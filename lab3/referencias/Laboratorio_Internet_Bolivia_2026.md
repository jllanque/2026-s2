# Laboratorio práctico — Enlaces Internacionales de Internet en Bolivia

**Temas:** fibra óptica · DNS · ASN · BGP · CDN  
**Modalidad:** trabajo individual o por parejas; **presentación oral individual**, incluso si la investigación se realizó en pareja.  
**Entrega:** un documento **Markdown (`.md`) por estudiante**, con las tablas de resultados, enlaces a las fuentes y observaciones breves. No se requieren diapositivas ni un informe extenso.

## Herramientas

- **Fibra:** [UIT Broadband Maps](https://bbmaps.itu.int/bbmaps/), [ATT SMI](https://smi.att.gob.bo/), [Submarine Cable Map](https://www.submarinecablemap.com/).
- **ASN/BGP:** [BGP.Tools](https://bgp.tools/), [HE BGP](https://bgp.he.net/), [LACNIC RDAP](https://rdap-web.lacnic.net/), [RIPEstat](https://stat.ripe.net/), [PeeringDB](https://www.peeringdb.com/).
- **DNS y latencia:** `dig`, `ping`, [Google Dig](https://toolbox.googleapps.com/apps/dig/), [DNSViz](https://dnsviz.net/), [DNSChecker](https://dnschecker.org/).
- **CDN:** [CDN Planet](https://www.cdnplanet.com/tools/cdnfinder/), [Cloudflare Radar Scanner](https://radar.cloudflare.com/scan), [PeeringDB](https://www.peeringdb.com/).

**Preparación:** en Debian/Ubuntu, `sudo apt update && sudo apt install dnsutils iputils-ping`. En todos los ejercicios, anotar fecha y enlace de consulta. Si no hay evidencia, escribir **«no verificable»** en lugar de adivinar.

## 1. Identificar infraestructura de fibra

**Herramientas y procedimiento:**

- [UIT Broadband Maps](https://bbmaps.itu.int/bbmaps/): acercar el mapa a Bolivia, activar las capas disponibles de redes de transmisión y seleccionar dos tramos reportados. Registrar los extremos o localidades visibles y el estado solo si la ficha lo indica.
- [ATT — Sistema de Monitoreo](https://smi.att.gob.bo/): localizar la sección de fibra óptica o los mapas publicados y contrastar uno de los tramos con la UIT. Anotar la fecha del documento.
- [TeleGeography — Submarine Cable Map](https://www.submarinecablemap.com/): seleccionar dos cables de las costas sudamericanas; abrir sus fichas y copiar una estación de amarre de cada uno.

**Sin diseñar ni dibujar rutas.** Identificar:

1. **Dos tramos de fibra terrestre** visibles en Bolivia o en sus fronteras: anotar país o ciudades de referencia, fuente y condición reportada si está indicada.
2. **Dos cables submarinos** de Sudamérica: anotar nombre, costa (Pacífico/Atlántico) y una estación de amarre publicada.
3. **Una diferencia:** ¿qué información ofrece el mapa y qué no permite demostrar sobre un ISP boliviano?

| Infraestructura identificada | Ubicación / extremos publicados | Fuente (URL) |
|---|---|---|
| Fibra terrestre 1 | | |
| Fibra terrestre 2 | | |
| Cable submarino 1 | | |
| Cable submarino 2 | | |

## 2. Identificar ASN y proveedores

**Herramientas y procedimiento:**

- [BGP.Tools](https://bgp.tools/): escribir el nombre comercial en el buscador, abrir la ficha del AS y anotar ASN, organización y prefijos. Comprobar que la entidad corresponde a Bolivia.
- [LACNIC RDAP](https://rdap-web.lacnic.net/): introducir dos ASN encontrados y contrastar el titular registrado con el nombre mostrado en BGP.Tools.
- [HE BGP Toolkit](https://bgp.he.net/): buscar el ASN del ISP y revisar las relaciones o redes adyacentes; anotar el upstream como **probable**, no como contrato demostrado.

1. Buscar los ASN de **ENTEL, AXS, COMTECO, COTAS y TIGO Bolivia** en BGP.Tools. Verificar **dos** en LACNIC RDAP.
2. Buscar un ASN relacionado con **Google/YouTube, Netflix, Meta/Facebook y TikTok/ByteDance**. En TikTok, distinguir red propia de proveedores externos; no asumir que existe un ASN único para todo el servicio.
3. Elegir **dos ISP bolivianos** e identificar **un posible upstream** de cada uno en BGP.Tools o HE BGP.

| Red / servicio | ASN encontrado | Titular o nombre visible | Upstream probable (solo ISP) |
|---|---|---|---|
| ENTEL | | | |
| AXS | | | |
| COMTECO | | | |
| COTAS | | | |
| TIGO | | | |
| Google / YouTube | | | — |
| Netflix | | | — |
| Meta / Facebook | | | — |
| TikTok / ByteDance | | | — |

## 3. Peering, tránsito y BGP

**Herramientas y procedimiento:**

- [BGP.Tools](https://bgp.tools/) o [HE BGP](https://bgp.he.net/): abrir el ASN elegido y consultar sus relaciones o vecinos. Anotar el tipo indicado por la fuente, si existe; de lo contrario marcar **incierto**.
- [PeeringDB](https://www.peeringdb.com/): buscar el nombre o ASN del ISP y de una red de contenido; revisar la política de peering, los IXP y las instalaciones declaradas. Coincidir en un IXP no prueba una sesión activa.
- [RIPEstat](https://stat.ripe.net/): introducir el ASN o un prefijo del ISP; consultar las vistas de anuncios/rutas y copiar un AS_PATH si aparece. La ruta es lógica, no geográfica.

1. Para **uno de los ISP** del ejercicio 2, consultar vecinos en BGP.Tools o HE BGP. Registrar **un tránsito probable y un peer probable**, si la fuente permite distinguirlos.
2. Buscar ese ISP y una red de contenido en PeeringDB. ¿Se declara presencia en algún IXP o política de peering?
3. En RIPEstat, observar **un prefijo** del ISP y anotar un AS_PATH si la herramienta lo muestra. Si no está disponible, indicar «no observable».

| Relación entre ASN | Tipo: tránsito / peering / incierto | Evidencia (herramienta/URL) |
|---|---|---|
| | | |
| | | |

**Responder en una frase:** ¿por qué aparecer como vecino BGP no prueba un contrato comercial ni una ruta física?

## 4. Identificar dominios y parámetros DNS

**Herramientas y procedimiento:**

- **Terminal Linux — `dig`:** ejecutar los comandos de abajo para cada dominio. Leer las respuestas `A`, `NS`, `MX`, `AAAA`, `SOA` y `TXT`; para registrar TTL utilizar `+noall +answer` (no `+short`).
- [Google Admin Toolbox Dig](https://toolbox.googleapps.com/apps/dig/): introducir un dominio, elegir `A` o `MX` y comparar con el resultado local.
- [Google Public DNS](https://dns.google/): consultar un dominio y observar los registros y TTL; una diferencia entre resolutores puede deberse a caché o políticas de respuesta.
- [DNSViz](https://dnsviz.net/): opcionalmente consultar uno de los dominios para observar delegación y DNSSEC; no interpretar la herramienta como prueba de ubicación del servidor.

Consultar obligatoriamente **`impuestos.bo`**, **`umsa.bo`**, **un dominio oficial de un ISP boliviano** y **un dominio de empresa privada boliviana** (verificar su nombre de dominio antes de usarlo).

Para cada dominio, obtener **A, NS y MX**; para **dos** de ellos, consultar además **AAAA, SOA y TXT**. Registrar al menos un **TTL** visible. Un registro ausente es un resultado válido.

```bash
dig impuestos.bo A
dig impuestos.bo NS +short
dig impuestos.bo MX +short
dig umsa.bo A +noall +answer
dig umsa.bo SOA +short
dig @1.1.1.1 impuestos.bo A
# Sustituir DOMINIO por los otros dos dominios elegidos:
dig DOMINIO A +noall +answer
dig DOMINIO NS +short
dig DOMINIO MX +short
```

| Dominio | A (IP o sin respuesta) | NS (uno) | MX (uno o ausente) | TTL (s) |
|---|---|---|---|---|
| impuestos.bo | | | | |
| umsa.bo | | | | |
| ISP elegido: | | | | |
| Empresa privada: | | | | |

**Responder:** ¿DNS permite saber por sí solo en qué país está alojado físicamente el servicio?

### Medición de latencia con `ping`

**Herramienta y procedimiento:** abrir la terminal de Debian/Ubuntu y ejecutar `ping -c 5 DOMINIO`. En el resumen final, copiar `packet loss` y el valor `avg` de `rtt min/avg/max/mdev`. Si `ping` no está instalado, instalar `iputils-ping`. Si el nombre no resuelve o el host filtra ICMP, dejar constancia sin inventar una latencia.

Realizar **5 solicitudes ICMP** a `impuestos.bo`, `umsa.bo` y al dominio del ISP elegido. Anotar el promedio RTT y la pérdida de paquetes. Si un destino no responde, registrar **«ICMP sin respuesta»**, sin concluir que el sitio está caído.

```bash
ping -c 5 impuestos.bo
ping -c 5 umsa.bo
ping -c 5 DOMINIO_ISP
```

| Dominio | IP resuelta | RTT promedio (ms) | Pérdida (%) | Observación |
|---|---|---:|---:|---|
| impuestos.bo | | | | |
| umsa.bo | | | | |
| ISP elegido | | | | |

**Observación:** el RTT incluye recorrido de ida y vuelta y procesamiento de red; no equivale a distancia física, latencia DNS ni rendimiento HTTP. La respuesta puede proceder de un proxy/CDN, no del servidor de origen.


## 5. CDN usadas por servicios populares en Bolivia

**Herramientas y procedimiento:**

- [BGP.Tools](https://bgp.tools/): buscar las redes de Netflix, Google, Meta y TikTok/ByteDance; registrar ASN y prefijos pertinentes. Un servicio puede utilizar más de una red o CDN.
- [PeeringDB](https://www.peeringdb.com/): buscar el ISP boliviano y la red de contenidos; revisar las ubicaciones e interconexiones declaradas. No equiparar presencia en un IXP con caché instalada.
- [CDN Planet — CDN Finder](https://www.cdnplanet.com/tools/cdnfinder/): introducir un dominio público de prueba y observar si detecta CDN; la ausencia de detección no descarta su uso.
- [Cloudflare Radar — URL Scanner](https://radar.cloudflare.com/scan): analizar una URL pública y revisar los dominios de recursos y los ASN reportados. El dominio principal no necesariamente aloja el video o las imágenes.
- [HE BGP](https://bgp.he.net/) o [RIPEstat](https://stat.ripe.net/): contrastar una relación o prefijo que resulte dudoso.
- Para **Cloudflare, Fastly y Akamai**, buscar el nombre en BGP.Tools/PeeringDB y registrar una característica de servicio observada en su información pública; no atribuir su uso a un ISP boliviano sin evidencia.

1. Investigar **Netflix, YouTube/Google, TikTok y Facebook/Meta** con BGP.Tools, PeeringDB y, cuando sea útil, CDN Planet o Cloudflare Radar Scanner.
2. Para **dos ISP bolivianos** elegidos, indicar si existe **evidencia pública de interconexión o distribución** con cada plataforma. Clasificar: **peering declarado**, **relación BGP observada**, **CDN/caché local confirmada por fuente específica**, o **no verificable**. No confundir estas categorías.
3. Explorar **Cloudflare, Fastly y Akamai**: encontrar su ASN o red en BGP.Tools/PeeringDB e identificar **un servicio o rasgo de su CDN**. Opcional: revisar Amazon CloudFront.

| Servicio | ISP boliviano 1: evidencia | ISP boliviano 2: evidencia | ¿Caché dentro del ISP demostrada? |
|---|---|---|---|
| Netflix | | | |
| Google / YouTube | | | |
| TikTok | | | |
| Facebook / Meta | | | |

| CDN comercial | ASN o red encontrada | Servicio / característica identificada |
|---|---|---|
| Cloudflare | | |
| Fastly | | |
| Akamai | | |

**Pregunta final (2 frases):** ¿qué datos adicionales harían falta para afirmar que un ISP boliviano dispone de caché Netflix o Google dentro de su red? ¿Por qué una IP de CDN no demuestra su ubicación física?

## 6. Entrega y presentación individual

**Presentación oral individual (máximo 10 minutos por estudiante), con apoyo del documento Markdown:**

1. **Fibra:** dos tramos y dos cables identificados.
2. **ASN/BGP:** dos ISP, upstreams y relaciones observadas.
3. **DNS y ping:** comparación de dos dominios bolivianos y latencias medidas.
4. **CDN:** matriz de Netflix, Google, TikTok y Meta por ISP, con conclusiones y límites de evidencia.

**Entrega:** guardar el archivo como `Laboratorio_Enlaces_Internacionales_Apellido_Nombre.md`. Incluir las tablas completas, una sección **Observaciones** con 3–5 conclusiones breves y las URL de evidencia. En parejas se puede investigar conjuntamente, pero cada estudiante entrega su propio documento y expone individualmente. No se pide desarrollar rutas de fibra ni demostrar contratos privados.

### Observaciones finales

Escribir de 3 a 5 observaciones sobre los resultados: diferencias entre fuentes, casos no verificables, relaciones de tránsito/peering y latencias. Cada afirmación relevante debe tener una fuente o indicar que es una inferencia.

**Nota metodológica:** los datos de mapas, ASN, DNS y CDN pueden cambiar. Distinguir hechos publicados, observaciones e inferencias es parte del ejercicio.
