# Datos de Murcia

El archivo [murcia-landmarks.geojson](murcia-landmarks.geojson) es el catálogo de referencia para los hitos urbanos. Se acompaña de [documentación de los lugares](../docs/lugares-murcia.md), con prioridades y estado del modelado. Sus coordenadas proceden de elementos OSM identificados explícitamente; la geometría nula de Casa Cerdá indica que su anclaje está pendiente. Los puntos de referencia no son entradas ni accesos hospitalarios.

`murcia.json` es un extracto derivado de OpenStreetMap. **© OpenStreetMap contributors**.

- Licencia de los datos y de este extracto: **Open Database License (ODbL) 1.0**.
- Información y atribución: https://www.openstreetmap.org/copyright
- Licencia completa: https://opendatacommons.org/licenses/odbl/1-0/
- Área: latitudes 37.922 a 38.047; longitudes -1.22 a -1.0925 (unos 11,2 × 13,9 km: de Thader y Nueva Condomina a La Arrixaca y El Palmar, y de Alcantarilla y el Polígono Industrial Oeste de San Ginés a Puente Tocinos, Monteagudo y los carriles Juanete y Manresa). En el juego, X de −7633,4 a 3553,5 m y Z de −7013,2 a 6901,8 m.
- Origen de coordenadas: latitud 37.984, longitud -1.133.
- Coordenadas locales: metros aproximados, X hacia el este y segundo eje hacia el sur. Proyección equirectangular local.
- Fecha y sello de la base de datos OSM: campos `fetchedAt` y `osmTimestamp` del JSON.
- Cruces peatonales: nodos OSM en `crossings`, con sus identificadores, coordenadas y etiquetas resumidas. La actualización separada registra `streetFeaturesTimestamp` y `streetFeaturesFetchedAt`. `marked` solo se activa cuando las etiquetas indican marcas; no se pintan cruces no marcados por defecto.
- Generación: `scripts/fetch-murcia.mjs`, con consulta Overpass documentada en el propio script.
- Recuadro y origen: la ficha de la ciudad, `cities/murcia.json` (MOD01), el único sitio donde están escritos, junto con las capas, los ficheros y la caché de descargas. `fetch-murcia.mjs` los guarda en `murcia.json` (`bbox`, `origin`, `bounds`). Para regenerarlo todo: `npm run city -- murcia` (o `--out=<carpeta>` para no tocar estos ficheros), que ejecuta los scripts en orden y termina con un informe de control; el orden, las opciones y cómo añadir otra ciudad están en [cities/README.md](../../cities/README.md). Los identificadores de los edificios del Catastro (desde 9 000 000 000) se numeran en el orden del GML, así que cambian al cambiar el recuadro: `murcia-enclosures.json` se regenera con ellos.

Las huellas y los nombres proceden de OSM, salvo los edificios con `source: "catastro"` (ver abajo). Las alturas sin `heightSource` se toman de las etiquetas OSM de altura o de plantas. Las demás llevan `heightSource`:

- `catastro`: plantas sobre rasante del Catastro.
- `catastro-area`: superficie construida entre huella.
- `kind`: altura por defecto según el tipo OSM.
- `landmark`: tabla de monumentos.
- `type`: mínimo por tipo de monumento.

`measured: false` indica una estimación. Ninguna altura es una medición de campo. Este formato conserva vías, huellas, zonas verdes y eje del río; no conserva todos los metadatos ni las restricciones de giro de OSM. De las relaciones multipolígono conserva las de edificio y las de suelo urbano (ver abajo). Su finalidad es recreación y juego, no navegación real.

El fichero está disponible directamente en `/data/murcia.json` en la versión publicada. La atribución del mapa permanece visible durante el juego.

- Niveles de las vías: `layer` y `tunnel` de OSM, añadidos por `scripts/enrich-levels.mjs` (sello en `levelsTimestamp`). El juego eleva un puente solo si pasa sobre otra vía para coches. Los puentes sobre el río, que solo cruzan caminos del cauce, quedan a nivel de calle. Las rampas de acceso bajan al 7 %.
- Glorietas: `roundabout: true` en las vías con `junction=roundabout` o `circular` de OSM (876 vías), añadido por `scripts/enrich-roundabouts.mjs` (sello en `roundaboutsTimestamp`; `fetch-murcia.mjs` ya lo conserva). El juego mantiene cada anillo a una sola cota y rellena la isleta de los que quedan elevados ([dosier 34](../../docs/dosier/34-rotondas-multinivel.md)).
- `murcia-places.json`: nombres de barrios y pedanías (nodos OSM `place`) y lugares del catálogo, para la etiqueta de zona y el mapa. Generado con `scripts/fetch-places.mjs`. **© OpenStreetMap contributors**, ODbL 1.0.
- Alturas del Catastro: **© Dirección General del Catastro**, datos abiertos INSPIRE Buildings (municipio 30030, Murcia, y los vecinos que toca el recuadro) del servicio ATOM https://www.catastro.hacienda.gob.es/INSPIRE/buildings/ATOM, con reconocimiento de la fuente. `scripts/enrich-heights.mjs` (con los GML de Murcia, Molina de Segura, Alcantarilla, Las Torres de Cotillas y Alguazas: los municipios de `catastro` en `cities/murcia.json`) lee `numberOfFloorsAboveGround` de cada `BuildingPart` (EPSG:25830, convertidas a WGS84), toma la parte que cubre la mayor fracción de cada huella OSM sin altura etiquetada (mínimo 30 % cubierto) y fija `height = plantas × 3,1 + 0,5` m. Marca esos edificios con `heightSource: "catastro"` y registra `heightsSource` y `heightsFetchedAt`. Las huellas siguen siendo las de OSM.
- Relaciones de edificio: `scripts/add-building-relations.mjs` y `fetch-murcia.mjs` leen `relation[building][type=multipolygon]`.
  - **Huella**: un edificio por anillo exterior, con los anillos interiores (patios, el ruedo de la Plaza de Toros) en `holes`.
  - **Identificación**: `osm: "relation/<id>"` e `id` = 8 000 000 000 + id de relación × 10 + anillo.
  - **Duplicados**: las vías que eran a la vez edificio y miembro exterior se sustituyen por la relación.
  - **Orden**: el script va antes de `enrich-heights.mjs`. Sello en `buildingRelationsTimestamp`.
- Uso y año del Catastro: `use` (`residential`, `industrial`, `commercial`, `office`, `public`, `religious`, `agricultural`, `other`) y `year` (`dateOfConstruction`).
  - **Origen**: los pone `enrich-heights.mjs` en los edificios OSM, a partir del `Building` del Catastro que cubre al menos el 30 % de la huella. `add-catastro-buildings.mjs` los pone en los añadidos.
  - **`religious`**: sale de OSM (tipo o nombre del edificio de culto), porque el Catastro no tiene ese uso.
- Alturas de monumentos: `scripts/monument-heights.mjs` se ejecuta al final y aplica `scripts/landmark-heights.json`.
  - **La tabla**: da la altura y, en su caso, la torre (`towerHeight`), cada una con su fuente y la marca `estimated`.
  - **Mínimos por tipo**: iglesias a 12 m, ermitas a 8 m, teatros y auditorios a 15 m, mercados a 12 m y estadios a 12 m. Todos son estimaciones.
  - **Complejos**: `enrich-heights.mjs` da a los hospitales y a las huellas de 2.000 m² o más las plantas más altas que alcanzan el 20 % de la huella.
  - **Sin dato del Catastro**: los edificios OSM toman superficie construida entre huella (1–7 plantas) o una altura por tipo (2 plantas por defecto, 1 para marquesinas y casetas, 7 m para naves).
  - **Más información**: detalle, fuentes y cifras en `docs/dosier/18-datos-monumentos.md`.
- Edificios que faltan en OSM: `scripts/add-catastro-buildings.mjs` añade, después de `enrich-heights.mjs`, el anillo exterior de cada `Building` del Catastro que OSM no cubre. Lee los ZIP de Murcia (30030), de Molina de Segura (30027), que ocupa el norte de la franja oeste y la esquina junto al Mirador de Agridulce, y desde MAP02 de Alcantarilla (30005: el enclave del oeste y la mitad occidental del Polígono Industrial Oeste), Las Torres de Cotillas (30038) y Alguazas (30007), con dos rincones del noroeste. Omite las ruinas (`condition: ruin`) y un solar ya derribado de El Palmar. Descarta los que comparten más del 30 % de su huella con un edificio OSM, los que tienen menos de la mitad con plantas sobre rasante (sótanos, patios), los de menos de 20 m² (casetas, transformadores), los absurdamente esbeltos (altura mayor que 6 veces el lado del cuadrado de igual área) y los que cruza el eje de una senda o un paso peatonal (salvo túneles y pasajes). Si el eje de una calzada los cruza o se acerca a menos de 1,2 m, se recorta el frente hasta 1,3 m del eje en lugar de descartarlos; se descartan solo si el edificio queda a ambos lados del eje, si el corte llega más allá de la propia calle (2,5 m más) o si se queda con menos del 60 % de la huella ([dosier 37](../../docs/dosier/37-era-alta.md): OSM dibuja el eje de muchas calles de pedanía pegado a las casas de un lado). La altura es la de la `BuildingPart` que más huella cubre (`plantas × 3,1 + 0,5` m), y el tipo sale del uso (`currentUse`). Llevan `source: "catastro"`, `measured: true` e identificadores desde 9 000 000 000, por encima de los de OSM. Se simplifican con Douglas-Peucker a 0,3 m y se redondean a 0,1 m. Registra `catastroBuildingsSource` y `catastroBuildingsFetchedAt`. Volver a ejecutarlo sustituye los añadidos. Detalle y cifras en `docs/dosier/11-edificios-catastro.md`.
- Suelo urbano (`areas`), [dosier 29](../../docs/dosier/29-verde-urbano.md): `fetch-murcia.mjs` trae `park` (vías `leisure=park|garden` y `landuse=grass`) y `water`. `scripts/add-urban-ground.mjs` añade el resto sin volver a bajar el mapa (`node_modules/.bin/tsx scripts/add-urban-ground.mjs [--cache=overpass.json] [--dry-run]`, después de `enrich-levels.mjs`; volver a ejecutarlo sustituye lo añadido; sello en `urbanGroundTimestamp`).
  - **De OSM** (vías y multipolígonos): `wood` (`landuse=forest`, `natural=wood`), `scrub` (`natural=scrub|heath|grassland`, `landuse=meadow|greenfield`), `lawn` (`landuse=village_green|recreation_ground|allotments`, `leisure=dog_park`, relaciones `landuse=grass`), `cemetery`, `pitch` (césped), `clay` (tierra, arcilla), `court` (pistas duras), `track`, `playground`, `parking` (`amenity=parking` en superficie o sin `parking`), `median` (`area:highway=traffic_island`) y las relaciones de parque y jardín como `park` con `osm: "relation/<id>"`.
  - **Medianas sin área en OSM**: la franja entre dos calzadas de sentido único y sentido contrario (`trunk` a `tertiary`, a nivel del suelo) con ejes a menos de 25 m, o 40 m si pasa el tranvía entre ellas. Se corta en transectos cada 4 m, a 0,25 m de cada bordillo. `median` es césped; `gravel`, grava a menos de 3 m de una vía del tranvía (`murcia-transit.json`) y entre calzadas de `trunk`. Se para donde otra calzada, un edificio o un área de OSM la cruzan.
  - **Filtros**: fuera los bosques y matorrales de más de 25 ha o más de 20 m de desnivel (la sierra la pinta el sombreador del suelo), los anillos de menos de 30 m² y las áreas con más del 8 % sobre una calzada o un edificio. Los edificios enteros dentro de un área van en `holes`, con los anillos interiores.
  - **No se añaden**: `landuse=construction|brownfield` (el suelo desnudo ya es tierra), `leisure=sports_centre` (rodea edificios; sus pistas van aparte), aparcamientos subterráneos, en altura o en la calzada, y `area:highway` distinto de isleta (1 área en el recuadro).

## Pantallas acústicas: `murcia-barriers.json`

- Generación: `node scripts/fetch-barriers.mjs`, que lee el recuadro de `murcia.json` (`bbox`) y consulta Overpass (primero overpass.openstreetmap.fr). **© OpenStreetMap contributors**, ODbL 1.0.
- Contenido: polilíneas en metros locales del juego, `height` (m sobre la calzada o el suelo), `type` (`transparent`, `metal`, `concrete`), `road` (vía OSM que bordean) y `evidence`: `osm` (mapeada en OSM; hoy ninguna en el recuadro), `prensa-contrato-A30-2024` (corredor, longitud total de 5 km y altura de 4,5-6 m según la prensa; los tramos concretos son estimados) o `heuristica` (todo estimado).
- Los tramos estimados siguen el borde exterior de la calzada donde hay edificios a menos de 70 m (MU-33) o 40 m (resto de autovías). No son mediciones: ver `docs/dosier/10-pantallas-acusticas.md`.
- Sin el fichero el juego funciona sin pantallas.

## Vallas de parques: `murcia-fences.json`

- Generación: `node scripts/fetch-park-fences.mjs [--extract=overpass.json]`, que lee `murcia.json` (recuadro, calles) y `murcia-transit.json` (vías del tranvía) y consulta Overpass (primero overpass-api.de) por `leisure=park|garden`, `barrier=fence|railing` y las puertas (`barrier=gate|entrance…`, `entrance=*`). Con `--extract` usa un extracto ya guardado con esos elementos. **© OpenStreetMap contributors**, ODbL 1.0. No regenera `murcia.json`.
- Contenido: `fences[]` (polilíneas en metros locales, `park`, `style`: `forja`, `malecon` o `generica`, y `evidence`: `osm` si la valla está mapeada junto al parque, `dosier-33` si es el contorno de un parque que las fotos del dosier dan por vallado) y `gates[]` (`point`, `width`, `kind`: `gate` de OSM, `path` donde entra un camino, `road` donde cruza una calle, y `fence`).
- El juego aparta cada valla de la acera y de las calzadas y la corta en las puertas (`src/park-fences.ts`, `docs/dosier/33-vallas-de-parques.md`). Sin el fichero el juego funciona sin vallas.

## Cementerios y recintos cerrados: `murcia-enclosures.json`

- Generación: `node scripts/fetch-enclosures.mjs [--extract=a.json,b.json]`, que lee `murcia.json` y `murcia-transit.json`, consulta Overpass (cementerios, barreras, aparcamientos y puertas; con `--extract`, extractos guardados de Overpass o de la API 0.6 de OSM) y lee la ortofoto PNOA del IGN (`scripts/pnoa.mjs`, CC BY 4.0) dentro de los cementerios detallados. **© OpenStreetMap contributors**, ODbL 1.0. No regenera `murcia.json`.
- Contenido: `fences[]` y `gates[]` con el mismo formato que `murcia-fences.json` (estilos `tapia`, `muro-verja`, `malla` y `sepulturas`; `evidence`: `osm`, `orto` o `catastro-foto`), `cypresses[]` (puntos `[x, z]` de los cipreses vistos en la ortofoto) y `buildings[]` (`id` de un edificio de `murcia.json` dentro de un cementerio, `kind`: `gallery` o `pantheon`, `height`, `cemetery`).
- El juego suma las tapias a las vallas de parques (`src/park-fences.ts`), los cipreses a los árboles (`src/vegetation.ts`) y aplica `buildings` antes de construir la ciudad (`src/enclosures.ts`). Ver `docs/dosier/40-cementerios-y-recintos.md`. Sin el fichero el juego funciona como antes.

## Semáforos: `murcia-signals.json`

- Generación: `node scripts/fetch-signals.mjs`, que lee el recuadro de `murcia.json` (`bbox`) y consulta Overpass (primero overpass-api.de) por los nodos `highway=traffic_signals`, `crossing=traffic_signals` y `crossing:signals=yes`; añade los pasos de peatones con semáforo que `murcia.json` ya trae (`crossings[].signal`). **© OpenStreetMap contributors**, ODbL 1.0. No regenera `murcia.json`.
- Contenido: `signals[]` con `id` (nodo OSM), `point` (metros locales), `kind` (`junction` si es `highway=traffic_signals`, `crossing` si es un paso de peatones) y `direction` (`forward`/`backward` de `traffic_signals:direction`, o vacío). Solo nodos que están en una vía de `murcia.json`.
- El juego agrupa los nodos a menos de 35 m en un cruce y les da fases (`src/signals.ts`, `docs/dosier/35-trafico-y-semaforos.md`). Sin el fichero el juego funciona sin semáforos.

## Ferrocarril: `murcia-railway.json`

- Generación: `node scripts/fetch-railway.mjs [extracto-overpass.json]`, que lee el recuadro de `murcia.json` y consulta Overpass (primero overpass-api.de, con reintentos de espera creciente). **© OpenStreetMap contributors**, ODbL 1.0. No regenera `murcia.json`, que no guarda vías de tren.
- Contenido, en metros locales del juego:
  - `rails`: vías `railway=rail|light_rail|disused` y `railway=construction` de ferrocarril (la Murcia–Almería y el acceso de alta velocidad están así en OSM), con `bridge`, `tunnel` (o `covered`), `layer`, `cutting`, `embankment`, `electrified` (también el proyectado), `construction`, `kind`, `gauge`, `service`, `usage` y `name`;
  - `crossings`: pasos a nivel (`railway=level_crossing`) en nodos de esas vías;
  - `fences`: vallas y muros (`barrier=fence|wall`) que siguen una vía a menos de 25 m;
  - `platforms` (andenes `railway=platform`) y `taxis` (paradas `amenity=taxi`), con `fetchedAt` y `osmTimestamp`.
- Extracto del 23/11/2026: 74 vías (43 en obras, 44 en túnel), 0 pasos a nivel y 0 vallas en OSM.
- Uso: `src/city.ts` cruza vías y calles a distinto nivel (`src/railway.ts`, `layTracks`); `src/render/railway.ts` dibuja las vías; `src/render/landmarks.ts` dibuja las de la estación del Carmen a nivel, sus andenes, marquesinas y la parada de taxis ([dosier 20 § 8](../../docs/dosier/20-modelos-emblematicos.md)); `RailWalls` frena el coche. Dosier [39](../../docs/dosier/39-ferrocarril.md). Sin el fichero no hay vías y la estación conserva sus edificios.

## Relieve: `murcia-terrain.bin`

- Fuente: **Modelo Digital del Terreno MDT05 (paso de malla 5 m), © Instituto Geográfico Nacional**, licencia **CC BY 4.0** (https://creativecommons.org/licenses/by/4.0/). Obtenido del servicio WCS INSPIRE del IGN (https://servicios.idee.es/wcs-inspire/mdt, cobertura `Elevacion4258_5`, ETRS89 geográficas) para el mismo recuadro que el mapa, con 0,001° de margen.
- Generación: `node scripts/fetch-terrain.mjs [cobertura.asc]`. Remuestrea a una malla de 10 m en las coordenadas locales del juego, aplica una mediana de 110 m (quita terraplenes, trincheras, el cauce y restos de edificios: el juego construye esos elementos por su cuenta), suaviza los escalones de 1 m de la llanura y deja a nivel de ribera la franja del Segura y del Reguerón que dibuja el render del río.
- Formato: cabecera de 28 bytes (`MTR1`, `nx`, `nz` en Int32; `x0`, `z0`, paso y cota de referencia en Float32, little-endian) y `nx × nz` alturas Int16 en decímetros, por filas de z. y = 0 es la ribera del Segura junto al Puente Viejo (cota de referencia en la cabecera, unos 42,6 m). Malla de 1 123 × 1 396, unos 3,1 MB. Con `terrain.snap` de `cities/murcia.json` la petición se ajusta a la rejilla del primer extracto por los dos lados y se hace en trozos cortados en su borde oeste: el WCS redondea el número de columnas con su propia celda, y una petición de más de unas 2 400 columnas vuelve con otra celda y mueve todas las muestras (MAP02).
- Sin el fichero el juego funciona con el suelo plano.

## Horizonte: `murcia-horizon.bin`

- Fuente: **Modelo Digital del Terreno MDT25 (paso de malla 25 m), © Instituto Geográfico Nacional**, licencia **CC BY 4.0** (https://creativecommons.org/licenses/by/4.0/). Mismo servicio WCS INSPIRE que el relieve (https://servicios.idee.es/wcs-inspire/mdt), cobertura `Elevacion4258_25`, para el recuadro del mapa ampliado 10 km por cada lado (37,827–38,142 N, 1,339–0,974 W).
- Generación: `node scripts/fetch-horizon.mjs [cobertura.asc]`. Remuestrea a una malla de 100 m en las coordenadas locales del juego. Cada muestra es la **máxima** de 5 × 5 lecturas a 20 m: lo que se ve en el horizonte son crestas, y un peñasco tan estrecho como el de Monteagudo se hundiría con la media. Lee la cota de referencia (y = 0) de `murcia-terrain.bin`, que debe existir antes.
- Formato: cabecera de 28 bytes (`MHZ1`, `nx`, `nz` en Int32; `x0`, `z0`, paso y cota de y = 0 en Float32, little-endian) y `nx × nz` cotas Int16 en decímetros **sobre el nivel del mar**, por filas de z. Malla de 322 × 352, 227 KB. Máximo 1 064 m (Carrascoy); Cabezo de Monteagudo 105 m.
- Solo visual: el juego lo dibuja alrededor del mapa, cosido al borde del terreno, sin colisión. Sin el fichero el borde del mapa sigue llano hasta la calima.

## Huerta: `murcia-huerta.bin`

- Fuente: **SIGPAC, recintos de la campaña vigente, © Fondo Español de Garantía Agraria (FEGA)**, licencia **CC BY 4.0** (https://creativecommons.org/licenses/by/4.0/). El servicio declara como atribución al FEGA; los datos de la Región los mantiene la Comunidad Autónoma, pero la licencia y la atribución del servicio son las del FEGA. Teselas vectoriales GeoJSON de nivel 15 de https://sigpac-hubcloud.es (`/mvt/recinto@3857@geojson/15/{x}/{y}.geojson`), sin clave.
- Generación: `node scripts/fetch-huerta.mjs [carpeta-de-teselas]`. Baja las 195 teselas que cubren el recuadro de `murcia.json` (unos 170 MB; solo las que falten en la carpeta) y quita los duplicados por clave de recinto, porque cada tesela trae enteros los recintos que la tocan. Quedan 57 789 recintos.
- Clases (bits 0-2): 0 sin cultivo (zona urbana, viales, improductivo, agua, edificaciones), 1 cítricos (CI), 2 tierra arable y huerta (TA, TH), 3 frutales y otros leñosos (FY, FS, OV, VI…), 4 matorral, pastos y forestal (MT, FO, PR, PA). Bits 3-6: dirección de las filas, la del lado largo del recinto, en 16 pasos de 11,25°. Bit 7: variante «abandonada» o en barbecho, elegida por un *hash* del recinto (30 % de los cítricos y frutales, 45 % de la tierra arable, TH nunca; en el matorral, el 30 % es la variante más densa). Es una estimación: SIGPAC no dice qué parcelas están abandonadas.
- Formato: cabecera de 24 bytes (`HRT1`, `nx`, `nz` en Int32; `x0`, `z0`, paso en Float32, little-endian) y `nx × nz` bytes por filas de z, comprimidos con zlib (deflate). Celda de 4 m: 2 798 × 3 480. Unos 490 KB.
- El juego lo pinta en el sombreador del suelo, sin triángulos nuevos. Sin el fichero el suelo queda como antes.

## Acequias: `murcia-acequias.json`

- Generación: `node scripts/fetch-acequias.mjs`, con Overpass (primero overpass-api.de). **© OpenStreetMap contributors**, ODbL 1.0.
- Contenido: las vías `waterway=canal|ditch|drain` del recuadro que van a cielo abierto (sin `tunnel`, `culvert=yes`, `covered=yes`, `location=underground` ni `layer` negativo), con `osm`, `name`, `kind` (`mayor`, `menor`, `azarbe`, `brazal`, `canal`, `ditch`, `drain`, por el prefijo del nombre), `width` si OSM lo trae y la polilínea en metros del juego. 410 vías y 103 km. `regueron` guarda los identificadores y la geometría de las vías del Reguerón (`waterway=river`, `loc_name=Reguerón`), para reconocerlas entre los ríos de `murcia.json`.
- Anchos y profundidades por tipo en `src/acequias.ts` (acequia mayor 6 m, menor 2 m, azarbe 3,5 m, brazal 0,8 m). Son estimaciones del dosier 16, no mediciones.
- Sin el fichero el juego funciona sin acequias y con el Reguerón como antes.

## Árboles: `murcia-trees.json`

- Generación: `node scripts/fetch-trees.mjs` (Overpass, `node["natural"="tree"]` en el recuadro de `murcia.json`; con `--from=fichero.json` reutiliza una respuesta guardada). **© OpenStreetMap contributors**, ODbL 1.0.
- Contenido: cada árbol como `[x, z, taxón, leaf_type]` en metros del juego; el taxón es `species`, `genus` o `taxon` y `leaf_type` va en una letra (`b`, `n`, `m`, `l`), y los dos se omiten si faltan. 10 800 árboles; 47 con taxón (27 palmeras `Arecaceae`, el *Ficus macrophylla* de Santo Domingo, 3 plátanos…) y 397 con `leaf_type`. Unos 170 KB.
- El juego decide la especie al cargar (`src/vegetation.ts`): primero por el taxón, después por la tabla de calles y plazas documentadas en el dosier 23; si no, queda el árbol genérico. Sin el fichero quedan solo los árboles genéricos de parques y corredores.

## Misiones: `murcia-missions.json`

- Escritas a mano. Formato, tono y guiones en [dosier 38](../../docs/dosier/38-misiones.md), sección 7; tipos en `src/mission-scripts.ts`. Los lugares (`places`) llevan `lat`, `lon`, `source` y el punto de calle `road` que pone `npx tsx scripts/mission-routes.ts --write`.
- Calles cortadas (`closures`, en una misión o en una etapa): lista de `{ "street"?, "way"?, "from"?, "to"?, "note"? }`.
  - `street`: nombre OSM de la calle; se cortan todas sus vías transitables (sin distinguir mayúsculas ni tildes).
  - `way`: id OSM de una vía, o lista de ids.
  - `from` y `to`: van juntos y limitan el corte al tramo entre ellos. Cada uno es una clave de `places` (o de `draws`) o `[lat, lon]`.
  - Las de la misión duran toda la partida; las de una etapa, mientras esa etapa está activa. Al acabar la misión, de cualquier forma, se abren todas.
  - El juego pone vallas en cada boca del tramo y un coche de policía en cada extremo. El GPS, el tráfico y las patrullas lo rodean (`src/roads.ts`, `close`; `src/closures.ts`). `tests/mission-reach.test.ts` falla si un corte no cierra nada o aísla una parada.
  - Ejemplo:

```json
"closures": [{ "street": "Gran Vía Escultor Francisco Salzillo", "from": [37.98669, -1.13217], "to": [37.98454, -1.13185], "note": "Procesión" }]
```
