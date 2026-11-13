# Módulo 3. Sistemas Autónomos (ASN) e interconexión de Internet

**Curso:** Enlaces Internacionales de Internet  
**Material de lectura para estudiantes**  
**Exposición:** 20 minutos dentro de las dos horas teóricas. El laboratorio de 60 minutos es independiente.

## Objetivos

Comprender qué es un sistema autónomo, cómo se identifica y registra, cómo se relaciona con otros sistemas y cómo investigar los operadores bolivianos mediante fuentes públicas. Este módulo enlaza la infraestructura física estudiada en el módulo 1 con el enrutamiento BGP del módulo 4.

## 3.1 Internet como federación de redes

Internet no es una red única administrada por una autoridad central. Es una interconexión de redes administradas por organizaciones diferentes: proveedores de acceso, operadores de transporte, universidades, empresas, redes de contenido y centros de datos. Cada una puede decidir cómo conecta sus redes, a quién compra tránsito y con quién intercambia tráfico directamente. Para coordinar estas decisiones se emplean políticas de enrutamiento entre dominios.

Un **Sistema Autónomo (AS)** es un conjunto de redes IP operadas bajo una política de enrutamiento definida. Su identificador es un **Número de Sistema Autónomo (ASN)**. La propiedad de un ASN no implica que una organización posea todas las fibras por las que viaja su tráfico. Tampoco equivale a una empresa: una empresa puede operar varios ASN y diferentes entidades pueden coordinar un mismo AS bajo condiciones apropiadas.

La diferencia entre los tres conceptos es fundamental:

- **Red física:** fibras, equipos ópticos, routers, enlaces y centros de datos.
- **AS:** dominio lógico de política de enrutamiento.
- **ASN:** número que identifica ese dominio cuando intercambia información de rutas.

Por ejemplo, un ISP boliviano puede operar su propio ASN, arrendar capacidad de transporte hasta Chile y contratar tránsito IP a un operador internacional. El ASN del ISP identifica su política de enrutamiento, pero no describe por sí solo el recorrido geográfico de sus paquetes.

## 3.2 Numeración, ASN públicos y privados

Los ASN modernos se codifican en 32 bits. Históricamente se usaron ASN de 16 bits, por lo que muchas redes antiguas tienen números menores que 65536. Los ASN pueden representarse en notación decimal simple, como `AS64512`.

Los rangos reservados para uso privado son **64512–65534** y **4200000000–4294967294**. Son útiles para ciertos diseños internos y laboratorios, pero no deben anunciarse como identificadores de tránsito en la tabla global de Internet. Otros valores están reservados para fines específicos; por eso no basta con asumir que cualquier número no privado está disponible.

### ¿Quién asigna un ASN en Bolivia?

La coordinación mundial de números de Internet se realiza mediante IANA y los registros regionales de Internet (RIR). **LACNIC** administra recursos numéricos para Bolivia y gran parte de América Latina y el Caribe. La organización solicitante debe justificar su necesidad de interconexión y describir su política de enrutamiento y las redes con las que se conectará, conforme a la política vigente.

**Demostración web:** [Política de asignación de ASN de LACNIC](https://www.lacnic.net/pt/web/lacnic/manual-3) y [MiLACNIC Query](https://query.milacnic.lacnic.net/). Buscar un ASN permite diferenciar el titular registrado de los anuncios observados en Internet.

## 3.3 Prefijos IP y origen de rutas

Una red IP se anuncia mediante **prefijos**, por ejemplo `203.0.113.0/24` (dirección reservada para documentación). El ASN que origina un anuncio BGP se denomina **AS de origen**. Una organización puede tener varios bloques IPv4 e IPv6, y un mismo prefijo puede ser anunciado por más de un ASN en configuraciones específicas de multiorigen (MOAS).

La asignación de un bloque de direcciones, su autorización para originarlo y su visibilidad en BGP son hechos diferentes:

1. El registro RDAP/WHOIS informa a quién está asignado el recurso.
2. Los registros IRR y los objetos ROA de RPKI pueden describir o autorizar anuncios de ruta.
3. Los colectores BGP muestran anuncios efectivamente observados desde determinados puntos de Internet.

Un ASN asignado puede no anunciar prefijos globales en un momento dado. Por ello, las listas de ASN registrados y las de ASN visibles en BGP no necesariamente coinciden.

## 3.4 Relaciones económicas entre sistemas autónomos

### Tránsito IP

El tránsito IP es un servicio mediante el cual una red proveedora ofrece a otra conectividad hacia destinos de Internet más allá de sus propias redes y clientes. Normalmente existe una relación comercial de pago. El ISP cliente puede comprar tránsito a uno o varios proveedores (*multihoming*) para mejorar disponibilidad, capacidad y opciones de política de rutas.

### Peering

En el peering, dos redes acuerdan intercambiar tráfico destinado a sus redes y, habitualmente, a sus clientes, sin ofrecer por ello acceso completo a todo Internet. El peering puede ser sin liquidación económica entre las partes (*settlement-free*) o pagado. Puede realizarse por una interconexión privada o en un punto de intercambio.

### IXP

Un **Internet Exchange Point (IXP)** proporciona infraestructura compartida, habitualmente de capa 2, para que múltiples sistemas autónomos establezcan sesiones de intercambio. Estar conectado al mismo IXP no implica que todos los participantes intercambien tráfico entre sí: deben existir acuerdos y sesiones de enrutamiento. Los *route servers* simplifican el establecimiento de rutas entre múltiples participantes, pero no sustituyen las políticas comerciales.

### Tier 1, Tier 2 y Tier 3

Son etiquetas comerciales informales, no categorías oficiales de IANA o LACNIC. Habitualmente se denomina Tier 1 a una red que alcanza la Internet global sin comprar tránsito, mediante relaciones de peering; Tier 2 a redes que combinan tránsito y peering; y Tier 3 a redes de acceso que principalmente compran conectividad. Estas etiquetas simplifican una realidad más compleja y pueden cambiar según región y criterio utilizado.

## 3.5 Un ejemplo de interconexión desde Bolivia

Supongamos que un usuario de La Paz accede a un sitio alojado en una red de contenidos. El tráfico sale de la red de acceso de su ISP y puede tomar distintos caminos:

- Si la red de contenidos tiene caché dentro del ISP, parte del tráfico puede permanecer en la red local.
- Si ambos sistemas intercambian tráfico directamente en un IXP o interconexión privada, puede evitarse parte del tránsito internacional.
- Si no existe interconexión directa, el ISP puede alcanzar el destino a través de su proveedor de tránsito, utilizando enlaces nacionales o internacionales según la topología.

El camino real depende de la política BGP, de la disponibilidad de prefijos y de la infraestructura física. **Un AS_PATH no es un mapa geográfico**: muestra una secuencia de sistemas autónomos en el plano de control, no las ciudades ni los cables recorridos.

### Operadores bolivianos como casos de observación

Durante la clase se investigarán los registros públicos y la visibilidad de redes vinculadas a **ENTEL, AXS, COMTECO, COTAS y TIGO Bolivia**. No se fija aquí una lista estática de ASN o proveedores: las estructuras empresariales, anuncios y relaciones pueden variar y deben comprobarse en el momento de la demostración.

Al examinar un operador conviene distinguir:

- Titular registrado del ASN y nombre comercial.
- Prefijos IPv4 e IPv6 visibles.
- ASN vecinos o rutas observadas en colectores públicos.
- Presencia declarada en puntos de intercambio y centros de datos.
- Posibles proveedores de tránsito y relaciones de peering, que no siempre pueden inferirse con certeza de una sola fuente.

## 3.6 Herramientas web: demostraciones en vivo

Estas demostraciones son ejemplos explicados por el docente. Los ejercicios del estudiante estarán en la guía de laboratorio independiente.

| Herramienta | Enlace | Qué muestra |
|---|---|---|
| MiLACNIC Query | https://query.milacnic.lacnic.net/ | Datos registrales y búsquedas de recursos numéricos |
| RDAP Web LACNIC | https://rdap-web.lacnic.net/ | Titular y detalles de un ASN o bloque IP |
| BGP.Tools | https://bgp.tools/ | ASN, prefijos y observaciones de rutas |
| Hurricane Electric BGP Toolkit | https://bgp.he.net/ | Prefijos y relaciones BGP estimadas |
| RIPEstat | https://stat.ripe.net/ | Visibilidad e historia de recursos de Internet |
| PeeringDB | https://www.peeringdb.com/ | Presencia y políticas de interconexión declaradas |
| RIPE Atlas | https://atlas.ripe.net/ | Mediciones de conectividad desde sondas distribuidas |

### Demo A: identificar un operador boliviano

Abrir [BGP.Tools](https://bgp.tools/) y buscar `ENTEL Bolivia` o `COMTECO`. Identificar un ASN asociado a la organización y observar su nombre, país y prefijos visibles. Corroborar el número en [MiLACNIC Query](https://query.milacnic.lacnic.net/) o [RDAP Web LACNIC](https://rdap-web.lacnic.net/). La coincidencia entre nombre comercial y razón social no siempre es literal.

### Demo B: observar los prefijos de un AS

Introducir el ASN encontrado en [RIPEstat](https://stat.ripe.net/) y revisar los prefijos anunciados, su visibilidad y los cambios históricos disponibles. Comparar la información con [BGP.Tools](https://bgp.tools/). Las diferencias pueden depender del momento de actualización y de los colectores utilizados.

### Demo C: investigar interconexiones

Buscar el mismo operador en [PeeringDB](https://www.peeringdb.com/). Si existe un registro, revisar instalaciones, IXP y política de peering. La ausencia de un registro no demuestra ausencia de interconexiones: PeeringDB es una base de datos mantenida por sus participantes.

### Demo D: comparar dos operadores bolivianos

Consultar, por ejemplo, ENTEL y AXS en [Hurricane Electric BGP Toolkit](https://bgp.he.net/) y [BGP.Tools](https://bgp.tools/). Observar diferencias de prefijos, vecinos BGP y conectividad visible, evitando afirmar que un AS es proveedor de otro únicamente por aparecer próximo en una representación gráfica.

## 3.7 Interpretación crítica de las fuentes

Las fuentes públicas responden preguntas distintas. RDAP y WHOIS informan sobre registros; BGP.Tools, RIPEstat y Hurricane Electric reúnen observaciones del plano de control; PeeringDB recoge datos declarados por operadores; RIPE Atlas mide conectividad desde puntos de observación concretos. Ninguna fuente por sí sola proporciona un inventario completo de contratos, capacidad instalada, rutas físicas o tráfico real.

Asimismo, **más ASN vecinos no significa necesariamente mejor servicio**, y una red con pocos ASN visibles puede disponer de alta capacidad, redundancia y excelente conectividad a contenidos locales. El rendimiento requiere mediciones adicionales de latencia, pérdida, congestión y disponibilidad.

## 3.8 Síntesis

Internet funciona gracias a la cooperación técnica y comercial de sistemas autónomos independientes. Un ASN identifica un dominio de política de enrutamiento; el tránsito y el peering describen formas diferentes de interconexión; los IXP facilitan el intercambio; y los anuncios BGP hacen posible que las redes conozcan rutas hacia los prefijos IP. Para comprender Internet en Bolivia es indispensable combinar registros de LACNIC, observaciones BGP y conocimiento de la infraestructura física estudiada en el módulo anterior.

**Próximo módulo:** BGP, atributos de ruta y políticas de selección.

## Referencias

- LACNIC, Manual de políticas: https://www.lacnic.net/pt/web/lacnic/manual-3
- LACNIC, MiLACNIC Query: https://query.milacnic.lacnic.net/
- PeeringDB, documentación: https://docs.peeringdb.com/
- RFC 1930, Guidelines for creation, selection, and registration of an Autonomous System: https://www.rfc-editor.org/rfc/rfc1930
- RFC 4271, Border Gateway Protocol 4: https://www.rfc-editor.org/rfc/rfc4271
- RFC 6996, Autonomous System (AS) Reservation for Private Use: https://www.rfc-editor.org/rfc/rfc6996
