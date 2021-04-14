# virtual-appraiser

## Description

Virtual-Appraiser emula la capacidad de búsqueda y análisis de información de los avaluadores inmobiliarios utilizando data histórica de ventas, datos abiertos y técnicas de web scrapping  basado en análisis de imágenes mediante técnicas de reconocimiento óptico de caracteres y de aprendizaje profundo (DeepLearning).

**Algoritmo**: Virtual-Appraiser

**Entrada**: dirección del inmueble (DI), Valor Compra M2 Inmueble (VCI), área (A), número de cuartos (NC), número de baños (NB), número de parqueadores (NG), número de pisos (NP), estrato (E), tipo de inmueble (TI), polígono de estaciones Transmilenio (PET), polígono de centros comerciales (PCC), polígono de parques (PP), polígono de CAIs (PC), dataset ventas de inmuebles (DVI), Página Web de Oferta de Inmuebles (PWOI).

**Salida**: Estimación de Precio M2 para Venta, PM2, ($/m2).

1. lat, lon = geodecodificador(DI)
2. PreciosM2InmSimilaresHistoricos = identificarInmSimilaresHist(lat, lon, DVI, A, NC,NB,NG,NP,E,TI, PET,PCC,PP,PC)
3. PreciosM2InmSimilaresOferta = identificarInmSimilaresOfertados(lat, lon, PWOI, A, NC,NB,NG,NP,E,TI, PET,PCC,PP,PC).
4. IntervaloConfianzaM2 = obtnerIntervaloConfianzaM2(PreciosM2InmSimilaresHistoricos, PreciosM2InmSimilaresOferta, NivelConfianza=0.95)
5. Si VCI dentro de IntervaloConfianzaM2:
6.     PM2 = valorSuperiorIntervaloConfianza(IntervaloConfianzaM2)
7. sino:
8.     PM2 = VCI * 1.03 * ValorizacionAnual
9. retornar PM2


**Metodología**:

Como se puede observar en el “Algoritmo: Virtual-Appraiser” propuesto, existen algunas rutinas las cuales constituyen en sí misma fases que deben trabajarse, unas más que otras. A continuación se describe las rutinas que deben profundizarse:

**1. geodecodificador**
Esta rutina, permite obtener los valores de latitud y longitud de un inmueble a partir de su dirección. Existen diferentes geodecodificadores gratuitos y pagos. Independiente de la elección, se debe garantizar la suficiente precisión para evitar que los atributos debidos a la ubicación de los inmuebles se vean fuertemente sesgados a valores que en la realidad no corresponden. Aun cuando los geodecodificadores de Google Maps, OpenStreet Map, etc. presentan buenos resultados, no siempre son precisos cuando las direcciones no son bien especificadas, por lo tanto siempre es importante contar con una segunda validación, por ejemplo, en esta fase puede dejarse que la geodocodifcación recaiga enteramente en el proveedor de geodecodificación  o puede aparecer un humano en el proceso. Una evaluación sistemática permitirá identificar el nivel de certeza de esta fase y esta a su vez permitirá cuantificar el nivel de confianza en la estimación del precio de M2. 

**2. identificarInmSimilaresHist**
Las transacciones históricas de inmuebles deben recopilarse mes a mes a partir de fuentes endógenas (propias de HABI) como exógenas (e.g., La Galeria Inmobiliaria vivienda nueva y usada). Esta data debe reflejar la actualidad y el pasado de los diferentes inmuebles vendidos en Bogotá y por esta razón constituye la base de conocimiento más importante. Sin embargo, existen algunos atributos de los inmuebles que pueden variar de mes a mes, como son la percepción de seguridad del vecindario, cercanía a parques, a centros comerciales y estaciones del mío, los cuales deben recuperarse mediante análisis geoespaciales automáticos utilizando para ello datos abiertos de polígonos de estos elementos y que hacen parte del desarrollo urbanístico de Bogotá. Teniendo presente la anterior, una vez caracterizado  el inmueble mediante técnicas de similitud de patrones (euclidianas y no euclidianas), se puede crear un ranking de los inmuebles más similares y finalmente retornar los valores de precio M2 de los 10 primeros (10 vendría a ser una parámetro heurístico que en últimas influenciará en el nivel de confianza final en la estimación de precio de M2).

**3. identificarInmSimilaresOfertados**
Cada día se recurre más a las plataformas digitales para ofertar inmuebles. Estas son una fuente de información que los avaluadores tienen en cuenta para obtener una idea del precio m2 de los inmuebles en una zona. Plataformas como www.fincaraiz.com,  www.metrocuadrado.com entre otras, permiten consultar gran parte de la oferta vigente en Bogotá (así como de otras ciudades). Las tecnologías de web scrapping permiten obtener información de los precios, pero estas plataformas están desarrollando técnicas para evitar el web scrapping textual automático. Por esto, en esta propuesta, y con el fin de obtener una solución escalable, se plantea trabajar con las imágenes capturadas mediante screenshots automáticos de las páginas web después de haber realizado consultas como cualquier humano, pero de manera emulada. La idea es poder reconocer los elementos de las páginas web utilizando técnicas de aprendizaje profundo y de reconocimiento óptico de caracteres y emular así el envío de eventos de mouse y teclado de forma automática para ir recopilando la información de los diferentes inmuebles que se ofertan a través de las plataformas digitales destinadas para tal propósito.

**4. obtnerIntervaloConfianzaM2**
Dado que el precio de M2 de una zona es una variable aleatoria y para no suponer ningún tipo de distribución probabilística a priori sobre dicha variable, proponemos (a partir de los precios de M2 tanto de los inmuebles vendidos como de los ofertados que más se parecen al inmueble de interés) obtener intervalos de confianza mediante bootstrapping (muestreo  aleatorio con reemplazo). Los intervalos de confianza ofrecen una manera de estimar, con alta probabilidad, un rango de valores en el que se encuentra el valor poblacional (o parámetro) de una determinada variable. Un intervalo de confianza del 95% indica que el valor poblacional se encuentra en un determinado rango de valores con un 95% de certeza. Como regla general, mientras mayor es el tamaño de la muestra, menor es la variabilidad para hacer la estimación del intervalo, lo que lleva a estimadores más precisos, pero esto dependerá de la cantidad de inmuebles similares que se logren encontrar mediante los algoritmos propuestos en identificarInmSimilaresOfertados y identificarInmSimilaresHist.

**5. Decisión final**
La decisión final del precio de M2 se ha propuesto de tal forma que HABI obtenga una ganancia por encima de la inversión. Por tal razón, si el valor de compra del m2 del inmueble se encuentra dentro del intervalo de confianza se propone que HABI trabaje con el valor superior del intervalo de confianza estimado. En caso contrario se propone que HABI obtenga al menos un 3% de ganancia sobre la valorización natural con el paso de los años que sufra el inmueble.

## How to run

1. Download this repo to a folder on your G-Drive
2. Download model from this URL, uncompress and locate in the same folder as mentioned in (1).
3. Open 'ElAlgoritmoEsCorrecto.ipynb' with Google Colab
4. Set dataPath variable according  your environment
5. Just run 'All Cells' on Google Colab and get your results

