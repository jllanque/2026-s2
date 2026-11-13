# Módulo 2. Sistema de Nombres de Dominio (DNS)

**Curso:** Enlaces internacionales de Internet  
**Material:** Guía de estudio del alumno  
**Exposición y demostraciones:** 20 minutos, dentro de las 2 horas teóricas del curso  
**Laboratorio:** independiente (1 hora para el conjunto de módulos); aquí no se incluyen ejercicios evaluables.

## 1. ¿Por qué DNS es fundamental para Internet?

Los equipos se comunican mediante direcciones IP, pero las personas suelen utilizar nombres como `www.google.com`. El **Domain Name System (DNS)** es una base de datos distribuida y jerárquica que permite asociar nombres con distintos tipos de información: direcciones IPv4 e IPv6, servidores de correo, servidores de nombres y políticas de seguridad. DNS no transporta la página web: proporciona información que permite localizar servicios; después intervienen IP, TCP o QUIC, TLS y HTTP, según el servicio.

Para un ISP boliviano, DNS afecta tanto a la experiencia del usuario como a la utilización de los enlaces internacionales. Un resolutor lento o distante puede añadir tiempo antes de establecer una conexión. Además, algunos servicios de distribución de contenidos utilizan respuestas DNS para orientar a los clientes hacia determinados nodos. Por eso, una respuesta DNS no es necesariamente idéntica para todos los usuarios del mundo.

**Idea clave:** una consulta DNS y la posterior conexión al servidor son operaciones diferentes; pueden recorrer rutas distintas y terminar en sistemas autónomos distintos.

## 2. Arquitectura distribuida y jerarquía

Un nombre plenamente calificado (FQDN), como `www.example.com.`, se interpreta desde la derecha: raíz (`.`), dominio de nivel superior (`com`), dominio (`example`) y subdominio (`www`). El punto final indica explícitamente la raíz, aunque normalmente se omite en el navegador.

Los actores más importantes son:

- **Stub resolver:** componente del sistema operativo o aplicación que formula consultas a un resolutor.
- **Resolutor recursivo:** recibe la consulta del cliente, responde desde caché si dispone de información vigente y, en caso contrario, obtiene los datos consultando otros servidores.
- **Servidores raíz:** informan sobre los servidores responsables de los dominios de nivel superior; no mantienen los registros A de todos los sitios web.
- **Servidores TLD:** indican los servidores autoritativos de un dominio delegado, por ejemplo bajo `.com` o `.bo`.
- **Servidores autoritativos:** proporcionan respuestas basadas en la zona DNS que administran.

### Secuencia simplificada de resolución

```text
Usuario en Bolivia
      |
      v
Stub resolver del equipo
      |
      v
Resolutor recursivo (ISP o público)
      | ¿Existe respuesta vigente en caché?
      |--- Sí ---> Respuesta al cliente
      |
      | No
      v
Servidor raíz (referencia al TLD)
      |
      v
Servidor TLD (referencia al autoritativo)
      |
      v
Servidor autoritativo (respuesta)
      |
      v
Resolutor almacena según TTL y responde
      |
      v
Cliente inicia conexión IP al destino
```

El esquema representa el caso general. En la práctica, el resolutor suele conservar en caché referencias a los servidores raíz, TLD y autoritativos; también puede usar mecanismos de minimización de consultas y resolver registros encadenados.

## 3. Registros DNS principales

| Tipo | Función | Ejemplo conceptual |
|---|---|---|
| A | Nombre a dirección IPv4 | `www.example.com` → `192.0.2.10` |
| AAAA | Nombre a dirección IPv6 | `www.example.com` → `2001:db8::10` |
| NS | Servidores de nombres de una zona | `example.com` → `ns1.example.net` |
| CNAME | Alias hacia otro nombre | `www` → `web.example.net` |
| MX | Servidores receptores de correo y prioridad | `example.com` → `mail.example.net` |
| TXT | Texto usado, entre otros fines, por SPF y verificaciones | Política o token |
| SOA | Información administrativa de zona | Servidor primario, serie y temporizadores |
| PTR | Resolución inversa de IP a nombre | `10.2.0.192.in-addr.arpa` |
| CAA | Autoridades de certificación autorizadas | Política de emisión |
| DS / DNSKEY / RRSIG | Componentes de DNSSEC | Cadena de confianza y firmas |

Las direcciones `192.0.2.0/24` y `2001:db8::/32` son rangos reservados para documentación, no resultados reales de una consulta.

**Precisión:** CNAME es un alias de nombre, no una redirección HTTP. Una respuesta A tampoco garantiza que un servidor web esté disponible en esa IP.

## 4. Caché, TTL y rendimiento

Cada registro se publica con un **Time To Live (TTL)**, expresado en segundos. El resolutor puede almacenar la respuesta durante ese tiempo, sujeto a su configuración y a las reglas del protocolo. Si un registro tiene TTL de 300 segundos, una copia recién obtenida puede mantenerse durante aproximadamente cinco minutos; una respuesta de caché posterior suele mostrar un TTL decreciente.

Una caché bien aprovechada disminuye consultas repetidas a servidores autoritativos, mejora los tiempos de resolución y reduce tráfico de control. Sin embargo, el ahorro en bytes de DNS suele ser pequeño comparado con el tráfico de video y descargas. Su impacto más relevante puede ser la latencia de establecimiento de conexiones y la elección del punto de entrega de contenido.

La llamada **propagación DNS** no es un mecanismo que empuje simultáneamente cambios a todos los servidores: normalmente describe la coexistencia temporal de respuestas antiguas y nuevas por cachés, TTL, delegaciones y políticas de los resolutores.

## 5. Transporte: UDP, TCP, cifrado y DNSSEC

DNS utiliza tradicionalmente UDP y TCP en el puerto 53. UDP es frecuente para consultas convencionales; TCP también es necesario en situaciones como respuestas grandes, truncamiento y ciertas operaciones entre servidores. EDNS(0) amplía capacidades del protocolo, incluido el tamaño de respuesta UDP anunciado.

**DNS-over-TLS (DoT)** cifra el transporte DNS, habitualmente sobre TCP/853. **DNS-over-HTTPS (DoH)** encapsula consultas DNS sobre HTTPS, habitualmente por 443. Ambos protegen el intercambio frente a observadores de red en el tramo cifrado, pero no hacen anónimo al usuario ni garantizan que el proveedor del resolutor sea confiable.

**DNSSEC** añade firmas criptográficas que permiten validar autenticidad e integridad de los datos DNS. No cifra las consultas ni oculta qué nombre se busca. La validación depende de una cadena de confianza desde la raíz y de la configuración del resolutor.

## 6. DNS, geografía y enlaces internacionales

El DNS puede participar en la selección de un servidor de contenidos. Una CDN puede devolver respuestas distintas en función del resolutor que pregunta, políticas de tráfico, disponibilidad o información geográfica. Con **EDNS Client Subnet (ECS)**, algunos resolutores pueden comunicar una porción de la red del cliente para mejorar ciertas decisiones de ubicación; esto también introduce consideraciones de privacidad.

Otra arquitectura usa **anycast**: la misma dirección IP se anuncia desde múltiples ubicaciones mediante BGP, y la red determina a cuál se dirige el tráfico. En ese caso, recibir la misma IP en Bolivia y otro país no significa necesariamente llegar al mismo centro de datos. No debe confundirse **DNS anycast** (distribución de servidores DNS) con **anycast de la CDN** (distribución de puntos de entrega HTTP).

### Ejemplo conceptual desde Bolivia

1. Un usuario solicita un dominio desde un ISP boliviano.
2. El equipo pregunta a un resolutor local del ISP o a un resolutor público.
3. El resolutor devuelve una dirección o una cadena de alias, posiblemente vinculada a una CDN.
4. La conexión HTTP(S) sale hacia esa dirección según las rutas IP y BGP disponibles.
5. El tráfico puede terminar en un nodo dentro de Bolivia, en un país vecino o en otro continente. La respuesta DNS por sí sola **no prueba** la ubicación física del servidor.

La ubicación del resolutor, la presencia de cachés dentro del ISP y las políticas de interconexión pueden influir en la experiencia y en el consumo de capacidad internacional.

## 7. Demostraciones web en vivo

Estas son demostraciones del docente, no ejercicios de laboratorio. Los resultados son dinámicos y pueden cambiar.

### Demo web A — Google Admin Toolbox Dig

**URL:** https://toolbox.googleapps.com/apps/dig/

Consultar `example.com` y comparar tipos **A**, **AAAA**, **NS** y **MX**. Mostrar que una consulta a un mismo nombre puede solicitar datos de clases distintas. Si un tipo no existe, explicar que una respuesta vacía no implica que el dominio no exista.

### Demo web B — Google Public DNS

**URL:** https://dns.google/

Consultar `cloudflare.com` y revisar tipo de registro, TTL y respuesta. Contrastar con el resultado obtenido en una consulta local con `dig` y explicar por qué no es obligatorio que ambos resolutores respondan exactamente igual.

### Demo web C — Resolución desde diferentes lugares

**URL:** https://www.whatsmydns.net/  
**Alternativa:** https://dnschecker.org/

Consultar registros A de un dominio público y observar posibles diferencias entre puntos de consulta. Advertir que estas herramientas miden respuestas desde sus propias sondas o resolutores y **no** representan necesariamente a todos los usuarios de cada país.

### Demo web D — DNSSEC visual

**URL:** https://dnsviz.net/

Analizar un dominio y explicar servidores autoritativos, delegación y cadena de confianza DNSSEC. Los indicadores de la herramienta describen la validación observada, no una garantía de seguridad integral del sitio.

### Demo web E — Diagnóstico de zona

**URL:** https://intodns.com/

Mostrar advertencias sobre delegación, NS, SOA y configuración general. Distinguir una recomendación operativa de un fallo comprobado.

### Demo web F — Exploración guiada del protocolo

**URL:** https://messwithdns.net/

Presentar la herramienta como recurso visual para entender registros y comportamiento de caché; las manipulaciones interactivas detalladas se reservan para el laboratorio.

## 8. Demostraciones con `dig` en Linux

`dig` (Domain Information Groper) es una utilidad de consulta DNS que permite especificar el nombre, el tipo de registro y el servidor consultado. Los comandos siguientes funcionan en Debian y Ubuntu con `dnsutils` o el paquete que proporcione `dig` en la distribución.

### Instalación y verificación

```bash
sudo apt update
sudo apt install dnsutils
command -v dig
dig -v
```

### Demo 1 — Consulta básica y lectura de respuesta

```bash
dig example.com A
```

Identificar `QUESTION SECTION`, `ANSWER SECTION`, `status`, `flags`, `Query time`, `SERVER` y TTL. **Query time** es tiempo de respuesta DNS, no latencia HTTP ni RTT hasta el servidor web.

### Demo 2 — Consultas IPv4, IPv6 y servidores de nombres

```bash
dig example.com A +short
dig example.com AAAA +short
dig example.com NS +short
dig example.com MX +short
```

El formato `+short` es útil para mostrar registros sin las secciones de diagnóstico.

### Demo 3 — Comparación de resolutores públicos

```bash
dig @1.1.1.1 cloudflare.com A
dig @8.8.8.8 cloudflare.com A
dig @9.9.9.9 cloudflare.com A
```

Comparar IP, TTL, tiempo de respuesta y servidor consultado. **No concluir que el resolutor con menor tiempo en una única prueba es siempre el mejor.** Si la red bloquea DNS externo por puerto 53, estas consultas pueden fallar sin que falle la navegación.

### Demo 4 — Recorrido de delegación desde la raíz

```bash
dig +trace example.com
```

Seguir las referencias desde raíz y TLD hasta los servidores autoritativos. `+trace` realiza consultas iterativas desde el equipo que ejecuta `dig`; no es un trazado de routers como `traceroute`. Algunas redes bloquean este tipo de consulta.

### Demo 5 — Consultar directamente a un servidor autoritativo

```bash
dig example.com NS +short
# Seleccionar un nombre de servidor devuelto y sustituirlo aquí:
dig @SERVIDOR_AUTORITATIVO example.com SOA +norecurse
```

Diferenciar respuesta autoritativa de una respuesta recursiva; la marca `aa` indica respuesta autoritativa cuando corresponde.

### Demo 6 — DNSSEC

```bash
dig @1.1.1.1 cloudflare.com A +dnssec
dig @1.1.1.1 cloudflare.com DNSKEY +dnssec
dig @1.1.1.1 cloudflare.com DS +dnssec
```

Buscar `RRSIG` y, cuando aparezca, la marca `ad`. **Importante:** la ausencia de `ad` en una respuesta no demuestra por sí sola que DNSSEC esté mal configurado. El registro DS de una delegación pertenece conceptualmente a la zona padre.

### Demo 7 — DNS inverso

```bash
dig -x 1.1.1.1 +short
```

Explicar que el registro PTR puede proporcionar un nombre asociado a una dirección IP, pero no constituye una prueba de titularidad, geolocalización ni identidad de un servicio.

### Demo 8 — Repetición, TTL y caché

```bash
dig @1.1.1.1 example.com A +noall +answer
sleep 3
dig @1.1.1.1 example.com A +noall +answer
```

Observar si cambia el TTL. En resolutores anycast o balanceados, las dos consultas pueden llegar a instancias distintas y el TTL no necesariamente disminuirá de forma lineal.

### Demo 9 — DNS sobre HTTPS desde terminal

```bash
curl -sS -H 'accept: application/dns-json' \
  'https://cloudflare-dns.com/dns-query?name=example.com&type=A'
```

Se consulta un servicio DoH que devuelve una representación JSON; este comando **no es `dig` usando DoH**, sino `curl` realizando una consulta HTTPS. Contrastar el puerto 443 con las consultas tradicionales de `dig` por el puerto 53.

## 9. Interpretación de resultados y limitaciones

- Una consulta DNS exitosa no demuestra que el sitio web responda.
- Una respuesta DNS distinta no implica necesariamente manipulación; pueden intervenir caché, balanceo y políticas CDN.
- `dig +trace` muestra delegación DNS, no el camino físico de los paquetes.
- Un registro A o AAAA identifica un destino lógico, no un país o centro de datos confirmado.
- Un resolutor público puede mejorar o empeorar la selección de contenido según el servicio y la red.
- DNSSEC autentica datos DNS, pero no sustituye HTTPS ni protege contra todas las amenazas.

## 10. Conexión con los siguientes módulos

Una vez obtenida la IP, es posible consultar qué **sistema autónomo (ASN)** anuncia el prefijo que la contiene. Luego, las herramientas BGP permiten observar rutas y relaciones de interconexión; finalmente, las herramientas CDN ayudan a investigar dónde se entrega el contenido. Así se conectan DNS, ASN, BGP, fibra óptica y distribución de contenidos en una misma arquitectura.

## 11. Referencias y documentación

- IETF RFC 1034 — Domain Names: Concepts and Facilities: https://www.rfc-editor.org/rfc/rfc1034
- IETF RFC 1035 — Domain Names: Implementation and Specification: https://www.rfc-editor.org/rfc/rfc1035
- IETF RFC 4033 — DNS Security Introduction and Requirements: https://www.rfc-editor.org/rfc/rfc4033
- IETF RFC 7858 — DNS over TLS: https://www.rfc-editor.org/rfc/rfc7858
- IETF RFC 8484 — DNS Queries over HTTPS: https://www.rfc-editor.org/rfc/rfc8484
- IETF RFC 7871 — Client Subnet in DNS Queries: https://www.rfc-editor.org/rfc/rfc7871
- Google Public DNS, ECS: https://developers.google.com/speed/public-dns/docs/ecs
- Cloudflare, arquitectura CDN: https://developers.cloudflare.com/reference-architecture/architectures/cdn/
- Cloudflare, herramientas DNS: https://developers.cloudflare.com/dns/reference/recommended-third-party-tools/
