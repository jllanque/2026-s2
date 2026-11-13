# Módulo 4. BGP: enrutamiento entre sistemas autónomos

**Curso:** Enlaces internacionales de Internet  
**Duración de exposición:** 20 minutos (dentro de las 2 horas de teoría y demostraciones)  
**Modalidad:** lectura para estudiantes y demostraciones en vivo; los ejercicios corresponden a una guía de laboratorio independiente de 60 minutos.  
**Enfoque:** conectividad internacional y operadores de Bolivia.

## Objetivos de aprendizaje

Al finalizar este módulo, el estudiante podrá explicar cómo BGP conecta sistemas autónomos, interpretar un anuncio de prefijo y su AS_PATH, diferenciar eBGP de iBGP, reconocer los principales atributos que influyen en la selección de rutas y comprender los límites de las herramientas públicas de observación BGP.

## 4.1. Por qué Internet necesita BGP

Internet es una federación de redes administradas por organizaciones diferentes. Cada red puede aplicar sus propias políticas técnicas y comerciales: elegir proveedores, contratar tránsito, intercambiar tráfico con otras redes y decidir qué prefijos anuncia. Un protocolo interior como OSPF o IS-IS calcula rutas dentro de un dominio administrativo; no está diseñado para expresar a escala global las relaciones y políticas entre miles de sistemas autónomos (AS).

**Border Gateway Protocol, versión 4 (BGP-4)** es el protocolo de enrutamiento interdominio utilizado para comunicar qué prefijos IP son alcanzables y por qué camino de sistemas autónomos se anuncian. Se describe habitualmente como un protocolo *path-vector*: además del prefijo de destino, transporta atributos, entre ellos el AS_PATH. BGP utiliza sesiones TCP (puerto 179) y actualizaciones incrementales después del intercambio inicial de rutas.

BGP **no transporta el tráfico de usuario**: intercambia información de rutas. Los paquetes de datos se reenvían utilizando las tablas de encaminamiento que resultan de las decisiones de los routers. Tampoco calcula necesariamente la ruta geográficamente más corta ni la de menor latencia; sus decisiones están condicionadas por políticas de cada AS.

### Relación con la fibra óptica

Una ruta lógica que aparece en BGP no identifica el cable submarino o la fibra terrestre exacta que transporta los paquetes. Dos proveedores pueden anunciar caminos AS diferentes y compartir, sin embargo, una misma infraestructura física o un mismo punto de falla. Por ello, **redundancia BGP** y **diversidad física** son propiedades distintas. Esta diferencia es fundamental para un ISP boliviano que compra capacidad internacional a varios operadores.

## 4.2. eBGP e iBGP

**eBGP (external BGP)** intercambia rutas entre routers pertenecientes a AS distintos. Por ejemplo, un ISP boliviano que contrata tránsito IP a un operador internacional puede establecer una sesión eBGP con ese proveedor.

**iBGP (internal BGP)** distribuye información BGP entre routers del mismo AS. Permite que varios routers de borde conozcan rutas aprendidas desde diferentes proveedores. A diferencia de un protocolo interior, iBGP no sustituye la necesidad de conectividad interna: los siguientes saltos anunciados deben poder alcanzarse mediante rutas internas o mecanismos equivalentes.

```text
               AS 65010 (ISP de ejemplo)
      +--------------------------------------+
      | R1 ----------- iBGP ------------ R2 |
      +--|--------------------------------|--+
         | eBGP                           | eBGP
         |                                |
    AS 65020 (Proveedor A)         AS 65030 (Proveedor B)
```

*Los ASN 65010, 65020 y 65030 son privados y se usan exclusivamente para ilustrar el concepto; no corresponden a operadores bolivianos reales.*

En redes pequeñas, los routers iBGP pueden establecer sesiones entre todos los participantes; en redes mayores se utilizan técnicas como **route reflectors** para reducir la cantidad de sesiones. La regla clásica de iBGP que impide volver a anunciar a otro vecino iBGP una ruta aprendida por iBGP explica por qué se necesitan estos diseños.

## 4.3. Prefijos, anuncios y AS_PATH

Un prefijo expresa un conjunto de direcciones IP en notación CIDR, por ejemplo `203.0.113.0/24` (bloque reservado para documentación). Cuando un AS origina un prefijo, otros AS pueden aprender su existencia mediante anuncios BGP. El prefijo anunciado no implica que el AS sea propietario del bloque: hay que distinguir titularidad de direcciones, autorización para originarlas y visibilidad del anuncio.

Supongamos que AS 65010 origina `203.0.113.0/24`. Si el anuncio llega a AS 65030 a través de AS 65020, el receptor puede observar un AS_PATH equivalente a `65020 65010` (desde su perspectiva). El AS_PATH ayuda a detectar bucles: un AS rechaza normalmente anuncios que ya contienen su propio número.

**Interpretación importante:** AS_PATH muestra la secuencia de AS presentes en el anuncio, no cada router, ciudad, fibra ni país atravesado. Además, algunas políticas de ingeniería de tráfico agregan repeticiones de un ASN (*AS-path prepending*), de modo que la longitud del AS_PATH tampoco representa una distancia física.

### Rutas por defecto y tabla completa

Un ISP puede recibir de su upstream una **ruta por defecto** (`0.0.0.0/0`, y en IPv6 `::/0`), una **tabla completa de Internet** (*full routing table*) o un conjunto parcial de rutas. La decisión depende de la arquitectura, la memoria disponible en los equipos, la necesidad de políticas de salida y el modelo de servicio. Recibir la tabla completa ofrece más información para decidir, pero no garantiza mejor rendimiento por sí mismo.

## 4.4. Atributos BGP y selección de rutas

| Atributo | Significado | Uso típico |
|---|---|---|
| `AS_PATH` | Secuencia de AS en el anuncio | Prevención de bucles e influencia en la preferencia |
| `NEXT_HOP` | Dirección IP del siguiente salto anunciado | Alcanzabilidad del camino |
| `LOCAL_PREF` | Preferencia interna del AS | Elegir salida entre varios upstreams; mayor suele preferirse |
| `MED` | Sugerencia de punto de entrada a otro AS | Influencia limitada, dependiente de políticas del vecino |
| `COMMUNITY` | Etiqueta para comunicar políticas | Solicitar tratamiento específico al proveedor |
| `ORIGIN` | Información sobre el origen BGP de la ruta | Criterio en ciertos procesos de selección |

El algoritmo concreto de selección varía entre implementaciones y configuraciones. En términos generales, un operador primero filtra rutas inaceptables y luego aplica políticas, preferencia local y otros criterios. **No existe una regla universal según la cual BGP siempre elija el AS_PATH más corto**. Una ruta con `LOCAL_PREF` superior puede ganar aunque su AS_PATH sea más largo.

### Ejemplo de política de salida

Un ISP en Bolivia recibe rutas por dos proveedores: A y B. Si para un destino establece `LOCAL_PREF=200` en las rutas de A y `LOCAL_PREF=100` en las de B, normalmente preferirá A, siempre que ambas rutas sean válidas y comparables dentro de su proceso de selección. El operador podría tomar esa decisión por costo, calidad del servicio o capacidad disponible.

### Tráfico entrante y saliente

El **tráfico saliente** se controla principalmente mediante las rutas que el propio ISP selecciona. El **tráfico entrante** depende de las decisiones de redes externas. El ISP puede intentar influir mediante anuncios selectivos, comunidades ofrecidas por proveedores o *AS-path prepending*, pero no puede garantizar que todas las redes remotas elijan el camino deseado. En particular, el prepending no sustituye acuerdos comerciales ni asegura balanceo.

## 4.5. BGP y enlaces internacionales de Bolivia

Un operador boliviano puede tener sesiones BGP con proveedores de tránsito internacionales y con redes de contenido o pares locales. Para comprender su conectividad se deben separar cuatro planos:

1. **Físico:** por dónde viaja la fibra y qué enlaces terrestres o submarinos intervienen.
2. **Transporte:** quién suministra capacidad Ethernet, longitudes de onda o circuitos entre puntos.
3. **IP/BGP:** qué ASN intercambian rutas y qué políticas aplican.
4. **Comercial:** quién vende tránsito IP, peering, transporte o colocación.

Una misma empresa puede prestar varios de estos servicios, pero no son equivalentes. Un proveedor de tránsito IP puede utilizar transporte arrendado de terceros; a la inversa, comprar un circuito hasta un punto de presencia internacional no proporciona automáticamente acceso a todas las redes de Internet.

### Dos rutas no significan dos caminos físicos

Si el ISP anuncia sus prefijos por dos upstreams, aumenta la diversidad lógica. Sin embargo, ambos pueden compartir ductos, pasos fronterizos, estaciones de regeneración o sistemas submarinos. La resiliencia real exige revisar rutas físicas, puntos de presencia, alimentación eléctrica, equipamiento, SLA y condiciones contractuales. Las herramientas BGP públicas no prueban esa diversidad física.

## 4.6. Seguridad del enrutamiento: hijacking, route leaks y RPKI

Un **secuestro de prefijo** (*BGP hijacking*) ocurre cuando una red anuncia un prefijo que no está autorizada a originar, con posibles efectos sobre disponibilidad, interceptación o desvío del tráfico. Una **fuga de rutas** (*route leak*) ocurre cuando se propagan anuncios en contra de las políticas esperadas entre redes, por ejemplo al reenviar rutas aprendidas de un proveedor hacia otro proveedor sin autorización.

**RPKI (Resource Public Key Infrastructure)** permite publicar autorizaciones criptográficas de origen mediante **ROA (Route Origin Authorization)**. Una validación de origen puede clasificar un anuncio como *Valid*, *Invalid* o *NotFound*. `Invalid` indica incompatibilidad con las autorizaciones ROA observadas, no demuestra por sí solo un ataque intencional. `NotFound` significa que no existe una autorización aplicable conocida, no que el anuncio sea necesariamente malicioso.

La validación de origen RPKI **no autentica la secuencia completa de AS_PATH**. Complementos como filtrado de prefijos, políticas de exportación, buenas prácticas operativas y mecanismos de protección de rutas siguen siendo necesarios.

## 4.7. Demostraciones web en vivo

Estas demostraciones ilustran la teoría; no son ejercicios evaluables. Para mantener el enfoque nacional, el docente buscará los ASN vigentes de **ENTEL Bolivia, AXS, COMTECO, COTAS y TIGO Bolivia** en los registros públicos. Los nombres comerciales no siempre coinciden exactamente con los nombres legales de los titulares ASN.

### Demo A — Identificar un ASN boliviano y sus prefijos

**Sitios:** [LACNIC Tools](https://tools.milacnic.lacnic.net/) · [BGP.Tools](https://bgp.tools/) · [Hurricane Electric BGP Toolkit](https://bgp.he.net/)

Buscar un operador boliviano y observar su número de AS, organización registrada, prefijos anunciados y relaciones visibles. Contrastar la información administrativa de LACNIC con la observación BGP. Una relación mostrada por un explorador BGP puede ser inferida a partir de rutas observadas y no necesariamente confirma un contrato de tránsito o peering.

### Demo B — Observar AS_PATH desde distintas redes

**Sitios:** [RIPEstat](https://stat.ripe.net/) · [HE Looking Glass](https://lg.he.net/)

Introducir un ASN o prefijo previamente identificado y explorar su visibilidad desde colectores o puntos de observación. Mostrar que diferentes observadores pueden recibir anuncios con AS_PATH distintos. Un Looking Glass refleja la perspectiva de una red o router concreto, no una vista completa de Internet.

### Demo C — Analizar un cambio o evento BGP

**Sitio:** [RIPEstat](https://stat.ripe.net/)

Usar la vista histórica de anuncios de un prefijo boliviano para explicar que las rutas pueden cambiar por mantenimiento, fallas, políticas o eventos de seguridad. No atribuir una causa específica sin evidencias adicionales de la red y del momento del evento.

### Demo D — Verificar el estado RPKI de un anuncio

**Sitios:** [RIPEstat](https://stat.ripe.net/) · [Cloudflare RPKI Portal](https://rpki.cloudflare.com/)

Comparar prefijo, ASN de origen y estado de validación. Explicar la diferencia entre *Valid*, *Invalid* y *NotFound*. Si no se encuentra un anuncio apropiado para ilustrar las tres categorías, explicar los estados sin forzar un ejemplo real.

## 4.8. Relación con el siguiente módulo

En el Módulo 5 se profundizará en las herramientas web para analizar ASN, prefijos, conectividad y cambios de rutas, con especial atención a los operadores de Bolivia. BGP proporciona la información lógica; los mapas de fibra del Módulo 1 muestran infraestructura física; el DNS del Módulo 2 ayuda a identificar direcciones y servicios. La combinación de estas fuentes permite investigar la conectividad internacional, siempre reconociendo lo que cada fuente **no** puede demostrar.

## Glosario breve

- **AS:** dominio administrativo de enrutamiento.
- **ASN:** identificador numérico de un AS.
- **BGP:** protocolo de intercambio de información de alcanzabilidad entre redes.
- **eBGP / iBGP:** BGP entre AS distintos / dentro del mismo AS.
- **Prefijo:** conjunto de direcciones representado mediante CIDR.
- **AS_PATH:** atributo que registra los AS del anuncio.
- **Upstream:** proveedor de conectividad hacia otras redes.
- **RPKI / ROA:** infraestructura y autorización para validar el origen de anuncios.
- **Looking Glass:** interfaz para observar rutas desde la perspectiva de una red.

## Referencias técnicas

- [RFC 4271 — Border Gateway Protocol 4](https://www.rfc-editor.org/rfc/rfc4271)
- [RFC 7454 — BGP Operations and Security](https://www.rfc-editor.org/rfc/rfc7454)
- [RFC 6811 — BGP Prefix Origin Validation](https://www.rfc-editor.org/rfc/rfc6811)
- [RFC 7908 — Problem Definition of BGP Route Leaks](https://www.rfc-editor.org/rfc/rfc7908)
- [LACNIC — Portal institucional](https://www.lacnic.net/)

---

**Nota:** Este módulo es material de lectura para el alumno. La exposición de 20 minutos selecciona los conceptos principales y realiza demostraciones breves. La guía de laboratorio independiente de 60 minutos contendrá las actividades y ejercicios de los seis módulos.
