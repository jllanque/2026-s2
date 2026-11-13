# Módulo 1. Infraestructura física de los enlaces internacionales de Internet

**Curso:** Enlaces Internacionales de Internet  
**Material:** Lectura del estudiante  
**Duración de exposición del módulo:** 30 minutos (dentro de 120 minutos de teoría y demostraciones).  
**Laboratorio:** 60 minutos adicionales, en una guía independiente.  
**Ámbito:** Sudamérica, con especial atención a Bolivia.

## 1. ¿Cómo llega Internet de Bolivia al resto del mundo?

Internet no es una red única administrada por una sola empresa. Es una interconexión de redes que comparten protocolos y acuerdos para intercambiar tráfico. Antes de que los routers puedan decidir por dónde enviar paquetes IP, debe existir una infraestructura física capaz de transportar las señales: fibras ópticas, sistemas de transmisión, equipos de energía, estaciones, centros de datos y puntos de interconexión.

Un usuario en La Paz puede acceder a un servidor ubicado en Estados Unidos, Brasil o Europa. Sin embargo, la trayectoria de los datos no tiene por qué ser una línea recta entre ambas ciudades. Los paquetes atraviesan la red de acceso del ISP, su red troncal, una o más redes de transporte e interconexiones entre operadores. En ocasiones el contenido se obtiene de una caché cercana, sin llegar al servidor de origen. Esta última situación se estudiará en el módulo de CDN.

Es importante distinguir **ruta física** y **ruta lógica**. La primera describe por dónde pasa el medio de transmisión; la segunda describe las redes IP y los sistemas autónomos que encaminan los paquetes. Una ruta BGP no revela por sí sola todos los kilómetros de fibra ni las instalaciones físicas utilizadas.

```text
Usuario ─ Red de acceso ─ Troncal del ISP ─ Transporte internacional
                                                   │
                                    Punto de interconexión / PoP
                                                   │
                                      Tránsito IP o peering
                                                   │
                                           Red de destino
```

## 2. Fibra óptica: el medio principal de las redes troncales

La fibra óptica es un medio dieléctrico que guía luz a través de un núcleo de vidrio. En las redes de larga distancia se utiliza predominantemente **fibra monomodo**, diseñada para transmitir señales ópticas con bajas pérdidas y grandes anchos de banda. El cable que vemos instalado en postes o ductos no es únicamente fibra: incluye protección mecánica, refuerzos y cubiertas apropiadas para su entorno.

### 2.1. Atenuación, dispersión y presupuesto óptico

La **atenuación** representa la reducción de potencia a medida que la señal avanza. Se expresa habitualmente en dB/km y depende, entre otros factores, de la longitud de onda, los empalmes, conectores y curvaturas. Como referencia técnica, las fibras modernas pueden presentar pérdidas del orden de 0,2 dB/km en torno a 1550 nm; el valor real depende del tipo de fibra y de las condiciones del enlace.

La **dispersión cromática** provoca que componentes espectrales diferentes se propaguen con retardos ligeramente distintos. En enlaces de alta velocidad, la dispersión y los efectos no lineales condicionan el alcance y la capacidad. Los sistemas ópticos coherentes modernos emplean procesamiento digital de señales para compensar determinadas degradaciones.

El presupuesto óptico relaciona potencia transmitida, pérdidas, ganancias de amplificación y sensibilidad del receptor. Por ello, no basta con conocer la distancia entre dos ciudades para afirmar que una fibra puede soportar determinada capacidad.

### 2.2. Multiplexación DWDM

DWDM (*Dense Wavelength Division Multiplexing*) permite transportar varios canales ópticos por una misma fibra, utilizando distintas longitudes de onda. Cada canal puede emplear una modulación y una velocidad distintas. La capacidad agregada resulta de la combinación de canales, espectro disponible, calidad de transmisión y equipamiento.

Un sistema DWDM puede incluir transpondedores, multiplexores, amplificadores ópticos, ROADMs y receptores coherentes. Los **ROADMs** permiten reconfigurar ópticamente la dirección de determinadas longitudes de onda sin convertir todos los canales a señales eléctricas.

Conviene evitar una simplificación frecuente: **una fibra no tiene una velocidad fija universal**. La capacidad útil depende del diseño completo del sistema óptico.

### 2.3. Latencia de propagación

La luz se propaga por la fibra a una velocidad inferior a la que alcanza en el vacío. Como aproximación, puede utilizarse **200.000 km/s**, equivalentes a **5 microsegundos por kilómetro**.

Para un trayecto óptico de 2.000 km:

\[
 t \approx \frac{2.000\ \text{km}}{200.000\ \text{km/s}} = 0,010\ \text{s} = 10\ \text{ms}
\]

Esos 10 ms corresponden solamente a propagación en un sentido. La latencia real incluye longitud adicional del tendido, equipos, procesamiento, colas y otros segmentos. El tiempo de ida y vuelta (*RTT*) suele ser mayor que el doble de la propagación mínima calculada.

## 3. Arquitectura de una red internacional

Una conexión internacional puede componerse de varias capas físicas y comerciales:

1. **Red de acceso:** conecta hogares, empresas o estaciones móviles con el ISP.
2. **Red troncal nacional:** agrega el tráfico entre ciudades y nodos principales.
3. **Red terrestre transfronteriza:** conecta instalaciones de operadores en distintos países.
4. **Backhaul:** lleva capacidad entre una estación de amarre, un centro de datos o un punto de presencia.
5. **Cable submarino:** transporta señales entre estaciones de amarre situadas en costas diferentes.
6. **Punto de presencia (PoP):** instalación donde un operador dispone de equipos y puede entregar o recibir servicios.
7. **Interconexión IP:** intercambio de tráfico entre redes, mediante tránsito o peering.

Un cable submarino no ofrece automáticamente acceso a Internet: entrega conectividad óptica entre puntos. Para llegar a los destinos de Internet se requieren equipos, servicios de transporte y relaciones IP entre redes.

## 4. Redes de fibra óptica terrestres en Sudamérica

La infraestructura regional combina grandes troncales nacionales, corredores fronterizos, redes metropolitanas y conexiones hacia estaciones submarinas. La distribución responde a la geografía, la concentración de población, las rutas de transporte, la demanda de capacidad y las decisiones de inversión de los operadores.

Brasil dispone de una extensa red interior y de conexiones atlánticas; Chile y Perú ofrecen acceso a infraestructura del Pacífico; Argentina y Uruguay participan en corredores del Cono Sur. Las interconexiones terrestres entre países permiten combinar rutas y llegar a diferentes mercados de tránsito e interconexión. Sin embargo, **la existencia de una ruta en un mapa no confirma que un operador determinado tenga derecho a utilizarla**.

### Demostración web A: redes terrestres de la UIT

**Mapa:** [ITU Broadband Maps](https://bbmaps.itu.int/bbmaps/)  
**Información institucional:** [ITU Interactive Transmission Maps](https://www.itu.int/en/ITU-D/Technology/Pages/InteractiveTransmissionMaps.aspx)

En el mapa pueden consultarse capas de redes de transmisión y, según los datos disponibles, distinguir infraestructura operativa, proyectada o en construcción. La cobertura y actualización varían por país. El mapa permite apreciar cómo los corredores de fibra conectan ciudades del interior con fronteras y puntos costeros.

**Qué observar:** los ejes de conectividad de Brasil, Argentina, Chile, Perú y Bolivia; las conexiones fronterizas; y la diferencia entre una red troncal nacional y un sistema submarino. La interfaz y la disponibilidad de capas pueden cambiar.

## 5. Los cables submarinos del Atlántico y del Pacífico

Los sistemas submarinos son esenciales para el transporte de tráfico intercontinental. Un cable contiene fibras ópticas protegidas por varias capas mecánicas y, en sistemas repetidos, incorpora equipos sumergidos que amplifican la señal. Las estaciones de amarre disponen de instalaciones de alimentación, terminación y conexión con redes terrestres.

La costa atlántica sudamericana enlaza con redes internacionales a través de sistemas que conectan Brasil y otros países con América del Norte, Europa y África. La costa pacífica ofrece rutas hacia otras partes de América y, según el sistema, hacia otras regiones. Los sistemas difieren en propietarios, consorcios, fechas de entrada en servicio, estaciones y capacidad; no deben tratarse como una única infraestructura homogénea.

### Ejemplos de sistemas para reconocer

- **SAm-1:** sistema histórico de conectividad regional e internacional.
- **SAC (South American Crossing):** sistema regional de cable submarino.
- **Curie:** sistema de Google con presencia en la costa pacífica de Sudamérica.
- **Mistral:** sistema submarino del Pacífico sudamericano.

Los nombres se incluyen para estudiar sus fichas y trayectos publicados; su presencia en un mapa no demuestra que un ISP boliviano utilice alguno de ellos.

### Demostración web B: mapa de TeleGeography

**Mapa interactivo:** [Submarine Cable Map](https://www.submarinecablemap.com/)  
**Advertencias metodológicas:** [TeleGeography — preguntas frecuentes](https://www2.telegeography.com/submarine-cable-faqs-frequently-asked-questions)

Este mapa permite localizar sistemas submarinos y consultar sus estaciones de amarre y fichas descriptivas. Al comparar las costas atlántica y pacífica se aprecia que las rutas internacionales no son intercambiables y que el acceso al cable depende de la ubicación de las estaciones y de los enlaces terrestres que llegan hasta ellas.

**Limitación fundamental:** las líneas del mapa son representaciones estilizadas; no constituyen planos exactos del recorrido bajo el mar. Tampoco indican por sí mismas disponibilidad comercial o capacidad libre.

## 6. Bolivia: conectividad de un país sin litoral

Bolivia no dispone de estaciones de amarre submarino en su territorio. Por ello, la conectividad internacional puede depender de corredores terrestres hacia países vecinos, donde se accede a redes regionales, centros de datos, puntos de presencia y, cuando corresponde, sistemas submarinos.

Desde el punto de vista geográfico, es relevante estudiar las interconexiones con **Perú, Chile, Brasil, Argentina y Paraguay**. Cada corredor ofrece posibilidades distintas de distancia, resiliencia, puntos de entrega y mercado mayorista. No obstante, las rutas realmente contratadas por ENTEL, AXS, COMTECO, COTAS, TIGO u otros operadores deben verificarse con fuentes comerciales o técnicas específicas: no se deducen de una frontera ni de una línea dibujada.

Una ruta conceptual hacia un punto internacional podría ser:

```text
La Paz → Troncal nacional → Paso fronterizo → Red de país vecino
      → Centro de interconexión / PoP → Tránsito IP → Destino
```

La ruta física y el camino de sistemas autónomos pueden ser diferentes. Por ejemplo, un operador puede comprar capacidad de transporte a una empresa y tránsito IP a otra; el ASN del transportista puede no aparecer en el AS_PATH de BGP.

### Demostración web C: infraestructura boliviana

**Portal sectorial:** [ATT — Sistema de Monitoreo e Información](https://smi.att.gob.bo/)  
**Mapa histórico de fibra óptica:** [ATT — mapa de red troncal (PDF)](https://smi.att.gob.bo/images/SMI/FibraOptica/MapasSMI-Fibraoptica.pdf)  
**Contexto histórico:** [AGETIC — Estado de las TIC en Bolivia (PDF)](https://agetic.gob.bo/sites/default/files/2025-06/AGETIC-Estado-TIC_dep.pdf)

Los recursos de la ATT permiten examinar información publicada sobre despliegues nacionales. Los documentos históricos sirven para entender la evolución de la cobertura, **no** para asegurar el estado actual de cada tramo. La información nacional debe contrastarse con los mapas de la UIT y con fuentes de los operadores.

### Tres preguntas que debe poder responder un mapa

**¿Dónde hay infraestructura?** Puede mostrar rutas y nodos reportados.  
**¿Quién la opera?** Solo si el mapa o la fuente identifican al responsable.  
**¿Se puede contratar capacidad?** Generalmente requiere confirmación comercial y técnica independiente.

## 7. Servicios mayoristas: fibra, transporte y tránsito IP

En el mercado de telecomunicaciones suelen mezclarse términos que corresponden a servicios diferentes:

| Servicio | Qué proporciona | Qué no garantiza |
|---|---|---|
| Fibra oscura | Uso de hilos de fibra sin iluminación del proveedor | Equipos ópticos, capacidad IP o Internet |
| Longitud de onda (*wavelength*) | Canal óptico entre puntos definidos | Acceso a todos los destinos de Internet |
| Transporte Ethernet / circuito | Conectividad entre dos puntos de entrega | Rutas IP globales |
| Tránsito IP | Alcance a destinos de Internet mediante una red proveedora | Propiedad de la fibra subyacente |
| Peering | Intercambio de tráfico entre redes según acuerdos | Acceso universal a Internet |

Un ISP boliviano puede adquirir un circuito de alta capacidad hasta un PoP extranjero y contratar allí tránsito IP. También puede comprar un servicio integrado que combine transporte y tránsito. Esta distinción es importante para interpretar costos, SLA, redundancia y responsabilidades en caso de fallas.

En las unidades siguientes se estudiará cómo los **ASN y BGP** permiten identificar relaciones de conectividad IP, pero sin confundirlas con la propiedad de cables o la topología óptica.

## 8. Resiliencia y diversidad de rutas

Tener dos proveedores no significa necesariamente disponer de dos rutas físicas independientes. Ambos pueden utilizar el mismo ducto, poste, puente, paso fronterizo, estación de amarre o cable submarino. Esos elementos compartidos representan **puntos únicos de falla**.

La resiliencia requiere estudiar la diversidad de rutas en varios niveles: acceso local, troncal nacional, cruces fronterizos, transporte regional, estaciones, cables y puntos de entrega. También importan la alimentación eléctrica, el mantenimiento, la gestión de fallas y los tiempos de reparación.

Un SLA puede establecer disponibilidad y tiempos de respuesta, pero no sustituye una auditoría de diversidad física. La documentación pública ayuda a formular preguntas; la validación requiere información del proveedor y, cuando corresponda, evidencias de ingeniería.

## 9. Capacidad, congestión y latencia: conceptos distintos

La **capacidad** expresa la tasa máxima de transporte de un enlace o servicio bajo determinadas condiciones. La **utilización** describe cuánto de esa capacidad se emplea. La **congestión** aparece cuando la demanda supera los recursos disponibles en algún punto, produciendo colas, pérdidas o degradación. La **latencia** mide el tiempo de transferencia y está condicionada por propagación, procesamiento y espera.

Un enlace de 100 Gbit/s no tiene necesariamente menor latencia que uno de 10 Gbit/s: la distancia y la ruta pueden dominar. A la inversa, una ruta corta puede funcionar mal si presenta congestión. Por eso, al comparar enlaces internacionales deben considerarse capacidad, latencia, pérdida, jitter, disponibilidad y diversidad, no solamente los Gbit/s contratados.

## 10. Lo que los mapas públicos pueden —y no pueden— demostrar

| Observación | Interpretación responsable |
|---|---|
| Una línea de fibra cruza una frontera | Existe o se ha reportado un corredor; debe verificarse estado y titularidad |
| Un cable llega a un país costero | Existe una estación o sistema asociado; no garantiza capacidad para un ISP específico |
| Dos rutas parecen separadas | La independencia física real requiere confirmación |
| Una ciudad aparece como nodo | Puede existir infraestructura, pero no necesariamente un punto de entrega comercial |
| Un ASN anuncia prefijos desde Bolivia | Es evidencia de conectividad lógica, no de qué cable físico transporta los paquetes |

## 11. Síntesis del módulo

La conectividad internacional se construye sobre una cadena de infraestructura: redes de acceso, troncales terrestres, interconexiones fronterizas, transporte regional, estaciones de amarre y cables submarinos. Sobre esa infraestructura operan redes IP autónomas que intercambian rutas y tráfico mediante acuerdos comerciales.

En Sudamérica, la geografía de los corredores terrestres y las costas atlántica y pacífica condiciona las opciones de conectividad. Para Bolivia, las interconexiones terrestres son especialmente importantes. Comprenderlas permite analizar con mayor rigor los servicios de transporte internacional, las rutas BGP, la redundancia y los costos de un ISP.

**Idea central:** un mapa físico explica dónde puede viajar una señal; no demuestra por sí solo qué camino toma un paquete IP, quién vende el servicio ni cuánto cuesta.

---

## Enlaces de consulta y demostraciones

1. [UIT — Broadband Maps](https://bbmaps.itu.int/bbmaps/)
2. [UIT — Interactive Transmission Maps](https://www.itu.int/en/ITU-D/Technology/Pages/InteractiveTransmissionMaps.aspx)
3. [TeleGeography — Submarine Cable Map](https://www.submarinecablemap.com/)
4. [TeleGeography — Submarine Cable FAQs](https://www2.telegeography.com/submarine-cable-faqs-frequently-asked-questions)
5. [ATT Bolivia — SMI](https://smi.att.gob.bo/)
6. [ATT — Mapa histórico de fibra](https://smi.att.gob.bo/images/SMI/FibraOptica/MapasSMI-Fibraoptica.pdf)
7. [AGETIC — Estado de las TIC en Bolivia](https://agetic.gob.bo/sites/default/files/2025-06/AGETIC-Estado-TIC_dep.pdf)

*Nota: los enlaces conducen a recursos de terceros; las capas, interfaces y disponibilidad pueden cambiar. Los datos históricos no deben interpretarse como inventarios actuales.*
