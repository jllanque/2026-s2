O# Laboratorio — Enlaces Internacionales de Internet en Bolivia

**Fecha de consulta de las fuentes:** noviembre de 2026 (RIPEstat muestra datos al 19/11/2026 16:00; Submarine Cable Map indica «Last updated Nov 17, 2026»)
**ISP elegidos para los ejercicios 2, 3 y 5:** AXS Bolivia (AS26210) y COMTECO (AS27839)
**Dominio de ISP para el ejercicio 4:** comteco.com.bo
**Dominio de empresa privada:** bnb.com.bo (Banco Nacional de Bolivia)

---

## 1. Infraestructura de fibra

| Infraestructura identificada | Ubicación / extremos publicados | Fuente (URL) |
|---|---|---|
| Fibra terrestre 1: La Paz – Patacamaya (Entel) | Extremos: La Paz y Patacamaya. Estado reportado: Operational. Operador según la ficha: «La Empresa Nacional de Telecomuncaciones SA» (tal como figura en la ficha). Distancia mostrada: 85.09 (la ficha no muestra la unidad) | https://bbmaps.itu.int/bbmaps/ |
| Fibra terrestre 2: Desaguadero – Ilo, «Fibra Óptica Soberana al Pacífico» (Entel) | Desaguadero (frontera Bolivia–Perú) hasta Ilo (Perú). Condición reportada: inaugurada el 4 de septiembre de 2020. Fuente de prensa; no la localicé en el mapa de la UIT | https://dplnews.com/?p=68506 |
| Cable submarino 1: South Pacific Cable System (SPCS)/Mistral (Pacífico) | Estación de amarre: Arica, Chile. Otras publicadas: Valparaíso (Chile), Salinas (Ecuador), Puerto San Jose (Guatemala) y Lurin (Perú). RFS agosto de 2021, 7,300 km; propietarios América Móvil (Claro) y Telxius | https://www.submarinecablemap.com/submarine-cable/south-pacific-cable-system-spcsmistral |
| Cable submarino 2: South Atlantic Cable System (SACS) (Atlántico) | Estación de amarre: Fortaleza, Brasil (el otro extremo es Sangano, Angola). RFS septiembre de 2018, 6,165 km; propietario Angola Cables | https://www.submarinecablemap.com/submarine-cable/south-atlantic-cable-system-sacs |

**Contraste con la ATT:** el sitio [ATT SMI](https://smi.att.gob.bo/) (sección «Mapas del sector», pestaña «Fibra óptica») muestra una red de fibra nacional con trazados en el eje La Paz – Oruro, donde queda Patacamaya, lo que es coherente con el tramo La Paz – Patacamaya de la UIT. Sin embargo, la imagen no identifica operador, extremos ni estado de cada trazado, por lo que **no permite confirmar ese tramo en concreto**. El propio mapa advierte que es información referencial, elaborada con datos reportados por los operadores a la ATT, y que no certifica disponibilidad de servicio en un punto determinado. Fecha del documento: no encontrada en el sitio.

**Una diferencia (qué ofrece el mapa y qué no demuestra):**
Los mapas muestran que existe infraestructura entre ciertos puntos y quién la opera, o al menos dónde está. No demuestran qué ISP boliviano usa esa fibra, cuánta capacidad contrata, ni si sus rutas lógicas BGP pasan por ella. Un ISP puede comprar tránsito a un proveedor que a su vez arrienda transporte de terceros.

---

## 2. ASN y proveedores

| Red / servicio | ASN encontrado | Titular o nombre visible | Upstream probable (solo ISP) |
|---|---|---|---|
| ENTEL | AS6568 | EMPRESA NACIONAL DE TELECOMUNICACIONES SOCIEDAD ANONIMA (también «Entel S.A. - EntelNet») | AS12956 (Telxius), AS3356 (Lumen / Level 3), AS6762 (Telecom Italia Sparkle / Seabone) y AS266757 (Satelital Telecomunicaciones S.A.C.), según la lista «Upstreams» de BGP.Tools (https://bgp.tools/as/6568); probables, no contractuales |
| AXS | AS26210 | AXS Bolivia S. A. | AS12956 (Telxius, Telefónica Global Solutions) |
| COMTECO | AS27839 | Comteco Ltda | AS28024 (Nuevatel) y AS6568 (Entel), según IPinfo. Nuevatel también figura como peer, por eso la relación es incierta |
| COTAS | AS25620 | COTAS LTDA. (Cooperativa de Telecomunicaciones Santa Cruz) | AS12956 (Telxius), AS7195 (EdgeUno) y AS8167 (V.tal), según IPinfo |
| TIGO | AS27882 | Telefónica Celular de Bolivia S.A. (PeeringDB: «Tigo Bolivia») | no verificable: las fuentes consultadas no listan upstreams |
| Google / YouTube | AS15169 | Google LLC | — |
| Netflix | AS2906 | Netflix Streaming Services Inc. | — |
| Meta / Facebook | AS32934 | Facebook, Inc. (BGP.Tools: registrado a ARIN-THEFA-3, tipo Content, 193 prefijos IPv4 y 254 IPv6 originados) | — |
| TikTok / ByteDance | AS138699, AS396986 y AS150436 | TIKTOK PTE. LTD. (Singapur), Bytedance Inc. (EE. UU.) y Byteplus Pte. Ltd. | — |

**Notas**

- **TikTok no tiene un ASN único.** Hay al menos tres relacionados. AS138699 (TikTok Pte.) aparece como proveedor de tránsito de AS396986 (ByteDance Inc.) y de AS150436 (BytePlus). El contenido además puede servirse desde CDN de terceros, así que ninguno de estos ASN prueba por sí solo de dónde llega un video a Bolivia.
- **Verificado en [LACNIC RDAP](https://rdap-web.lacnic.net/):** AS26210 figura a nombre de AXS Bolivia S. A. (handle `BO-ACBS1-LACNIC`, asignación directa, registrado el 2002-07-25) y AS27839 a nombre de Comteco Ltda (handle `BO-COLT2-LACNIC`, asignación directa, registrado el 2006-06-22). Coinciden con el nombre que muestra RIPEstat para esos mismos ASN. URL: https://rdap-web.lacnic.net/rdap/autnum/26210 y https://rdap-web.lacnic.net/rdap/autnum/27839.
- Todo lo de upstream es **probable**, no contractual. Proviene de agregadores que infieren relaciones a partir de rutas observadas.

**Fuentes:**
https://ipinfo.io/AS6568 (ver páginas de prefijos), https://bgp.tools/prefix/2800:88:f::/48, https://ipinfo.io/AS27839/200.58.86.0/24, https://ipinfo.io/AS25620/201.222.64.0/19, https://www.peeringdb.com/asn/27882, https://ipgeolocation.io/browse/asn/396986, https://ipinfo.io/AS138699/103.136.222.0/23, https://www.bgpv6.com/export/asn/138699/info.json

---

## 3. Peering, tránsito y BGP

| Relación entre ASN | Tipo: tránsito / peering / incierto | Evidencia (herramienta/URL) |
|---|---|---|
| AS26210 (AXS) ← AS12956 (Telxius) | Tránsito probable | BGP.Tools, página del prefijo 2800:88:f::/48 lista a AS12956 como upstream: https://bgp.tools/prefix/2800:88:f::/48. Coincide con ipgeolocation.io (https://ipgeolocation.io/browse/asn/26210) |
| AS27839 (COMTECO) ↔ AS6568 (Entel) | Incierto: IPinfo lo lista como upstream, pero ambos están en el IXP PIT Bolivia | https://ipinfo.io/AS27839/200.58.86.0/24 y https://bgp.tools/ixp/PIT%20Bolivia |
| AS27839 (COMTECO) ↔ AS28024 (Nuevatel) | Incierto: aparece a la vez como upstream y como peer en IPinfo | https://ipinfo.io/AS27839/200.58.86.0/24 |
| AS26210 (AXS) ↔ AS27839 (COMTECO) | Peering probable en el IXP PIT Bolivia (ambos son miembros) | https://www.peeringdb.com/ix/3333. Estar en el mismo IXP no prueba una sesión activa |
| AS6568 (ENTEL) ← AS12956 (Telxius), AS3356 (Lumen) y AS6762 (Sparkle) | Tránsito probable (BGP.Tools los lista como upstreams) | https://bgp.tools/as/6568 |

**PeeringDB (ISP y red de contenido)**

- AXS Bolivia (AS26210): en el PIT Bolivia figura con puerto de 100G, política Open, IPv4 45.68.0.7 e IPv6 2801:11:a800:1::7 (https://www.peeringdb.com/ix/3333).
- COMTECO (AS27839): en el PIT Bolivia figura con puerto de 40G, política Open, IPv4 45.68.0.5 e IPv6 2801:11:a800:1::5 (https://www.peeringdb.com/ix/3333).
- Red de contenido: el export de PeeringDB del PIT Bolivia (https://www.peeringdb.com/ix/3333) lista 18 peers, 15 con política Open, 472G de capacidad total y 94 % con IPv6. Entre ellos hay dos redes que no son operadores locales de acceso: Alibaba Cloud CDN (AS24429, 40G) y EdgeUno International (AS64124, 100G); también aparecen Entel, Tigo (Telefónica Celular de Bolivia, 50G), Nuevatel, COTAS, Comteco y AXS. **Google, Netflix y Meta no figuran entre los 18 miembros.** Solo se revisó este IXP: no se descartan otros IXP ni interconexiones privadas.

**AS_PATH en RIPEstat:** la pestaña Overview confirma que `2800:88::/32` es anunciado por AS26210 (AXS Bolivia S. A.) y `190.11.88.0/21` por AS27839 (Comteco Ltda), ambos visibles como coincidencia exacta (datos al 19/11/2026 16:00; https://stat.ripe.net/resource/2800:88::/32 y https://stat.ripe.net/resource/190.11.88.0/21). AS_PATH: no verificable (en la pestaña Routing de RIPEstat no se encontró un AS_PATH visible).

**Respuesta en una frase:** Aparecer como vecino BGP solo indica que dos ASN intercambian anuncios de ruta; no revela si hay contrato de pago, y menos aún por qué fibra física viajan los paquetes, porque BGP describe el plano lógico y no la ruta geográfica.

---

## 4. Dominios, DNS y latencia

Los resultados de esta sección se obtuvieron ejecutando `dig` y `ping` desde WSL (Ubuntu) el 18/11/2026; los valores de TTL, IP y RTT dependen de la conexión y de la hora.

Comandos usados:

```bash
# impuestos.bo
dig impuestos.bo A +noall +answer
dig impuestos.bo NS +short
dig impuestos.bo MX +short

# umsa.bo
dig umsa.bo A +noall +answer
dig umsa.bo NS +short
dig umsa.bo MX +short
dig umsa.bo AAAA +noall +answer
dig umsa.bo SOA +noall +answer
dig umsa.bo TXT +noall +answer

# ISP elegido (COMTECO)
dig comteco.com.bo A +noall +answer
dig comteco.com.bo NS +short
dig comteco.com.bo MX +short
dig comteco.com.bo AAAA +noall +answer
dig comteco.com.bo SOA +noall +answer
dig comteco.com.bo TXT +noall +answer

# Empresa privada (BNB)
dig bnb.com.bo A +noall +answer
dig bnb.com.bo NS +short
dig bnb.com.bo MX +short

# Comparación con otro resolutor
dig @1.1.1.1 impuestos.bo A
```

Para registrar el TTL, es la segunda columna de la salida con `+noall +answer`.

| Dominio | A (IP o sin respuesta) | NS (uno) | MX (uno o ausente) | TTL (s) |
|---|---|---|---|---|
| impuestos.bo | sin respuesta (NXDOMAIN) | sin respuesta | sin respuesta | no aplica |
| impuestos.gob.bo (consulta adicional) | 170.82.244.41 | logan.ns.cloudflare.com (también kami.ns.cloudflare.com) | 10 antispam22.weblinksrl.com | 300 |
| umsa.bo | 200.7.161.56 | marte.umsa.bo (también jupiter.umsa.bo) | 1 aspmx.l.google.com (más 4 alternos de Google) | 2708 |
| ISP elegido: comteco.com.bo | 200.58.72.105 | ns.supernet.com.bo (también ns1.supernet.com.bo) | 20 comtecoRL.mail.protection.outlook.com | 10800 |
| Empresa privada: bnb.com.bo | sin respuesta (no hay registro A en el dominio raíz) | ns.bnb.com.bo (también cuatro servidores de Azure DNS) | 0 bnb-com-bo.mail.protection.outlook.com | no aplica |

Consulta realizada el 18/11/2026 desde WSL (Ubuntu) con `dig`. Para `impuestos.bo` también se probó el resolutor 1.1.1.1 (`dig @1.1.1.1 impuestos.bo A`), que respondió `status: NXDOMAIN`, con el SOA de la zona `.bo` (servidor `ns.nic.bo`). Un registro ausente es un resultado válido según la guía.

**Dos dominios con AAAA, SOA y TXT**

| Registro | umsa.bo | comteco.com.bo |
|---|---|---|
| AAAA | 2801:104:dc:a161::a56 (TTL 60) | sin respuesta |
| SOA | marte.umsa.bo. admdns.umsa.bo. serie 2022032365 (TTL 3600) | ns.comteco.com.bo. root.comteco.com.bo. serie 2026101200 (TTL 10800) |
| TXT | Verificaciones de Google y GlobalSign; SPF `v=spf1 a mx ip4:200.7.160.201 include:spf.mandrillapp.com include:_spf.google.com -all` (TTL 60) | Verificaciones de Google y Microsoft (`MS=`); SPF `v=spf1 ip4:200.58.64.120 ip4:200.58.64.234 include:spf.protection.outlook.com -all` (TTL 10800) |

**Lo que se puede leer de estos resultados (hechos observados)**

- UMSA gestiona sus propios servidores de nombres (`marte` y `jupiter` dentro de `umsa.bo`) y usa Google para el correo.
- El dominio de COMTECO delega su DNS a `supernet.com.bo`, que en LACNIC aparece vinculado a un bloque IPv6 de Comteco (https://ipregistry.co/AS27839/2803:9400::/48), y usa Microsoft 365 para el correo.
- BNB usa Azure DNS, un servidor propio y Microsoft para el correo.
- BGP.Tools confirma que 200.7.161.56 (UMSA) está dentro de 200.7.161.0/24, 200.7.160.0/22 y 200.7.160.0/20, todos originados por AS27828 con visibilidad alta (https://bgp.tools/prefix-selector?ip=200.7.161.56). También que 170.82.244.0/23 y 170.82.244.0/24 son originados por AS264858 con visibilidad alta; la captura corresponde a la IP 170.82.244.51 y no a la 170.82.244.41 que resolvió `dig`, pero ambas están en el mismo /24 (https://bgp.tools/prefix-selector?ip=170.82.244.51). Para 200.58.72.105 (COMTECO), la página de BGP.Tools devolvió un error de uso; una consulta posterior en RIPEstat confirmó que corresponde a AS27839 (Comteco Ltda) (https://stat.ripe.net/resource/200.58.72.105).

**Contexto:** las capturas de BGP.Tools muestran los ASN que originan esas IP (AS27828 para umsa.bo y AS264858 para impuestos.gob.bo); el laboratorio no exige identificar su titular, por lo que no se afirma aquí.

### Latencia con `ping`

```bash
ping -c 5 impuestos.bo
ping -c 5 umsa.bo
ping -c 5 comteco.com.bo
```

| Dominio | IP resuelta | RTT promedio (ms) | Pérdida (%) | Observación |
|---|---|---:|---:|---|
| impuestos.bo | no resuelve | no medido | no medido | `ping: impuestos.bo: Name or service not known`. Coincide con el NXDOMAIN de `dig`; no hay paquetes que perder porque nunca se envió ninguno |
| impuestos.gob.bo (consulta adicional) | 170.82.244.41 | ICMP sin respuesta | 100 | 5 paquetes enviados, 0 recibidos. No se concluye que el sitio esté caído: el host puede filtrar ICMP |
| umsa.bo | 200.7.161.56 | 2.897 | 0 | 5 de 5 respuestas; mín/máx 2.679/3.372 ms, mdev 0.245 ms; el DNS inverso muestra `www.umsa.bo`; TTL del paquete 60 |
| comteco.com.bo | 200.58.72.105 | 8.103 | 0 | 5 de 5 respuestas; mín/máx 7.185/10.734 ms, mdev 1.336 ms; el DNS inverso muestra `pacha2.supernet.com.bo`; TTL del paquete 56 |

Medido el 18/11/2026 desde WSL (Ubuntu) con `ping -c 5`, ejecutado desde mi equipo en La Paz. Para `impuestos.bo` no se registra latencia porque el nombre no existe; si se prueba `impuestos.gob.bo`, anotar aquí su resultado aparte.

**Observación:** los RTT son bajos (unos 3 ms hacia UMSA y unos 8 ms hacia COMTECO), pero eso no prueba dónde están los servidores. El RTT incluye ida y vuelta y procesamiento de red, y no equivale a distancia física, latencia DNS ni rendimiento HTTP. Lo que sí se puede decir como hecho observado es que ambos respondieron sin pérdida y que el nombre inverso de la IP de COMTECO pertenece al dominio `supernet.com.bo`, el mismo que aloja sus servidores DNS.

**Responder:** No. Un registro A o NS solo dice qué dirección IP corresponde a un nombre; la IP puede pertenecer a una CDN, a un proxy o a un servicio anycast, y su ubicación física requiere otras evidencias (registro del ASN, mediciones de latencia desde varios puntos, información del propio proveedor). El RTT tampoco equivale a distancia: incluye procesamiento y puede responder un proxy, no el servidor de origen.

---

## 5. CDN usadas por servicios populares en Bolivia

**Clasificación usada:** peering declarado / relación BGP observada / CDN o caché local confirmada por fuente específica / no verificable.

| Servicio | ISP boliviano 1 (AXS): evidencia | ISP boliviano 2 (COMTECO): evidencia | ¿Caché dentro del ISP demostrada? |
|---|---|---|---|
| Netflix | Netify detecta 2 direcciones de Netflix CDN dentro de AS26210 (observación de terceros). Peering declarado: Netflix no figura entre los 18 miembros del PIT Bolivia, donde AXS participa. Relación BGP observada: no consultada | Netify detecta 6 direcciones de Netflix CDN dentro de AS27839 (observación de terceros). Peering declarado: Netflix no figura entre los miembros del PIT Bolivia, donde COMTECO participa | No demostrada. Una detección de Netify es una heurística, no una confirmación del ISP o de Netflix |
| Google / YouTube | No aparece entre las redes con IP de Google Edge CDN que lista Netify. Google no figura entre los miembros del PIT Bolivia. No verificable | Netify detecta 9 direcciones de Google Edge CDN dentro de AS27839 (observación de terceros). Google no figura entre los miembros del PIT Bolivia | No demostrada |
| TikTok | No verificable (TikTok/ByteDance no figura entre los miembros del PIT Bolivia) | No verificable (igual) | No demostrada |
| Facebook / Meta | No verificable (Meta no figura entre los miembros del PIT Bolivia) | No verificable (igual) | No demostrada |

Fuentes: https://www.netify.ai/resources/cdn/netflix-cdn/country/bo y https://www.netify.ai/resources/cdn/google-cloud-cdn/country/bo.

**Contexto:** en esos mismos listados de Netify aparece mucho más contenido de Google en Entel (81 direcciones) que en COMTECO, lo que ilustra que la presencia varía mucho entre ISP, pero sigue siendo una observación de un tercero.

| CDN comercial | ASN o red encontrada | Servicio / característica identificada |
|---|---|---|
| Cloudflare | AS13335, Cloudflare, Inc. (BGP.Tools: tipo Content, registrado a ARIN-CLOUD14, 2734 prefijos IPv4 y 3114 IPv6 originados) | Según su página de red (https://www.cloudflare.com/network/): más de 335 ciudades en más de 125 países, más de 13,000 interconexiones de red y una latencia de 50 ms para el 95 % de los usuarios de Internet (cifras comerciales de Cloudflare) |
| Fastly | AS54113 (Fastly, Inc.; PeeringDB: tipo Content) | Plataforma de borde con CDN programable; su política pública de peering es selectiva y declara que no tiene backbone propio (https://www.fastly.com/jp/peering, https://peeringdb.com/asn/54113) |
| Akamai | AS20940, Akamai International B.V. (BGP.Tools: registrado en RIPE como `nl.akamai`, tipo Content, 3920 prefijos IPv4 y 756 IPv6 originados); también AS16625, Akamai Technologies Inc. | Su portal, visible en la ficha de BGP.Tools, presenta líneas de producto de nube (Akamai Cloud) y seguridad (Akamai Security) |

**Pregunta final (2 frases):**
Para afirmar que un ISP boliviano tiene una caché de Netflix o Google dentro de su red haría falta una fuente específica, como una declaración del ISP o del programa de caché, junto con evidencia técnica de que los servidores tienen direcciones del propio ISP y responden con latencias coherentes con una ubicación local. Una IP de CDN no prueba su ubicación física porque las CDN usan anycast, proxies y direcciones que cambian según el resolutor y el momento, y las bases geográficas de IP son estimaciones.

---

## Observaciones

1. **Los upstreams que encontré son inferencias.** AXS tiene un upstream (Telxius, AS12956) respaldado por BGP.Tools y por un segundo agregador, y ENTEL muestra cuatro upstreams en BGP.Tools (Telxius, Lumen, Sparkle y Satelital Telecomunicaciones); en cambio, COMTECO muestra a Nuevatel y Entel a la vez como upstream y como peer, lo que impide clasificarlos con certeza. Tomado de IPinfo y BGP.Tools; inferencia.
2. **Coincidir en un IXP no prueba tráfico.** AXS (100G) y COMTECO (40G) son miembros del PIT Bolivia según PeeringDB, lo que solo permite suponer un peering probable. El mismo IXP incluye a Alibaba Cloud CDN (AS24429) y EdgeUno International (AS64124), por lo que no todos los miembros son operadores bolivianos; Google, Netflix y Meta no figuran entre los 18. Fuente: https://www.peeringdb.com/ix/3333
3. **La infraestructura física y la lógica son planos distintos.** Entel aparece vinculada a salidas por Perú (Ilo–Lurín) y Chile (Arica), y hay fuentes que la describen conectando el Pacífico con el Atlántico, pero los mapas y BGP no dicen cuál usa cada ISP privado. Fuentes de la sección 1.
4. **Las evidencias de caché local son indirectas.** Netify detecta direcciones de Netflix y Google en ISP bolivianos, pero ninguna fuente que consulté confirma formalmente cachés dentro de AXS o COMTECO. Inferencia a partir de datos de terceros.
5. **Casos no verificables o sin respuesta:** `impuestos.bo` devolvió NXDOMAIN (no existe como dominio en la zona `.bo`), así que no se obtuvo A, NS ni MX; en cambio `impuestos.gob.bo` sí resuelve (IP 170.82.244.41, NS de Cloudflare y correo en `weblinksrl.com`) pero no respondió al ping (100 % de pérdida), lo que indica filtrado de ICMP o un host que no contesta, no necesariamente una caída. Que sus NS sean de Cloudflare solo dice quién aloja la zona DNS, no dónde está el servidor web. También quedan sin verificar el upstream de TIGO, la presencia de TikTok y Meta en los ISP elegidos y el AS_PATH (no verificable en RIPEstat).

**Nota metodológica:** la mayoría de los datos de ASN y relaciones los obtuve de agregadores públicos (IPinfo, ipgeolocation.io, BGP.Tools, HE BGP, PeeringDB, Netify) que pueden diferir entre sí y cambiar con el tiempo. Las afirmaciones de relaciones (upstream, peering) son probables, no contractuales.
