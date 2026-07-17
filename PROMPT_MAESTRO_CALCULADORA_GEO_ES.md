# Prompt maestro de reconstrucción - Calculadora Geo 1.0

> Copia desde **INICIO DEL PROMPT** hasta **FIN DEL PROMPT** y entrégalo a un agente de programación con acceso a Windows 11, Node.js y npm. Este documento es la especificación reproducible de la versión 1.0 entregada.

---

## INICIO DEL PROMPT

Actúa como arquitecto de software, desarrollador senior de Electron, especialista en geometría computacional, integración multimodal y seguridad de aplicaciones de escritorio. Construye desde cero **Calculadora Geo 1.0**, reproduciendo exactamente el producto descrito en este contrato. No entregues una maqueta, una demo, pseudocódigo ni archivos con marcadores pendientes. El resultado debe ser una aplicación completa, ejecutable, probada y empaquetada.

No hagas preguntas si una decisión ya está definida aquí. No cambies el catálogo, los nombres, las fórmulas, la disposición visual, la arquitectura, los valores predeterminados ni los nombres de los artefactos. No agregues telemetría, cuentas, inicio de sesión, actualizador, servicios en segundo plano, servidor LAN, firma digital ni certificados. La calculadora determinista es siempre la autoridad; la IA solo reconoce entradas o explica resultados.

### 1. Resultado obligatorio

Entrega una aplicación de escritorio para **Windows 11 x64**, en castellano, denominada **Calculadora Geo**, versión **1.0.0**, con estas características:

- Cálculo determinista de exactamente 31 figuras, formas y conversores.
- Cálculo directo y, donde se indica, inverso.
- Geometría 2D, sólidos 3D, Tesseract, 3-sphere (S³) y 4-simplex.
- Conversión dimensional de longitudes, áreas, volúmenes y magnitudes de cuarta potencia.
- Reconocimiento mediante imagen subida o cámara, con lectura de cotas, números, unidades y texto adherido al dibujo.
- Voz mediante dictado del sistema, grabación de micrófono o archivo de audio.
- Revisión humana obligatoria antes de aplicar datos reconocidos.
- Compatibilidad mediante API OpenAI-compatible con Ollama, LM Studio, llama.cpp y proveedores cloud opcionales.
- Funcionamiento matemático completo sin IA y sin Internet.
- Modo **Local estricto** activado de fábrica.
- Portable e instalador NSIS x64 deliberadamente sin firma digital.

La carpeta final de entrega será:

```text
F:\HERRAMIENTAS\CALCULADORA-GEO
```

Debe contener, como mínimo:

```text
Calculadora-Geo-1.0.0-x64-Portable.exe
Calculadora-Geo-1.0.0-x64-Instalador.exe
codigo-fuente\
LEEME.md
```

### 2. Pila tecnológica y versiones

Usa exactamente esta arquitectura:

- Node.js y npm.
- Electron `^43.1.1`.
- electron-builder `^26.15.3`.
- JavaScript moderno sin TypeScript.
- HTML y CSS nativos, sin React, Vue, Angular ni servidor web.
- Módulos ES en el renderer y motor matemático.
- CommonJS para `electron/main.cjs` y `electron/preload.cjs`.
- Pruebas con `node:test` y `node:assert/strict`, sin framework adicional.
- No añadas dependencias de ejecución. Las únicas dependencias declaradas serán las dos devDependencies anteriores.

El `package.json` tendrá:

```json
{
  "name": "calculadora-geo-desktop",
  "version": "1.0.0",
  "description": "Calculadora geométrica universal local para Windows 11 con reconocimiento multimodal e IA configurable.",
  "private": true,
  "type": "module",
  "main": "electron/main.cjs"
}
```

Scripts obligatorios:

```json
{
  "start": "electron .",
  "test": "node --test tests/*.test.mjs",
  "pack": "electron-builder --win dir",
  "dist": "electron-builder --win nsis",
  "portable": "electron-builder --win portable --config.win.artifactName=Calculadora-Geo-${version}-${arch}-Portable.${ext}",
  "release": "npm test && npm run dist && npm run portable"
}
```

Configuración exacta de electron-builder:

- `appId`: `local.calculadorageo.desktop`.
- `productName`: `Calculadora Geo`.
- `asar`: `true`.
- Incluir solo `electron/**/*`, `src/**/*` y `package.json`.
- Salida: `dist`.
- Objetivos Windows: `portable` x64 y `nsis` x64.
- Nombre NSIS: `Calculadora-Geo-${version}-${arch}-Instalador.${ext}`.
- NSIS no one-click, permitir cambiar directorio, crear accesos directos de escritorio e Inicio, nombre de acceso `Calculadora Geo`, no borrar AppData al desinstalar.
- No configurar firma, certificado, publisher ni Store.

### 3. Estructura exacta del proyecto

```text
calculadora-geo-desktop/
├─ electron/
│  ├─ main.cjs
│  └─ preload.cjs
├─ src/
│  ├─ ai/
│  │  └─ local-parser.js
│  ├─ math/
│  │  └─ engine.js
│  ├─ app.js
│  ├─ index.html
│  └─ styles.css
├─ tests/
│  └─ math.test.mjs
├─ package.json
├─ package-lock.json
├─ README.md
└─ ENTREGA.md
```

### 4. Contrato del motor matemático

Implementa en `src/math/engine.js` y exporta:

```js
FIELD_LABELS
CATEGORIES
CATALOG
LINEAR_UNITS
solve(shapeId, inputs = {}, changedField = '')
convertNumber(value, fromUnit, toUnit, power = 1)
formatNumber(value, locale = 'es-ES')
specFor(id)
```

Reglas generales:

- Acepta coma o punto decimal.
- Solo calcula con números finitos positivos.
- Un parámetro editable de una figura de un parámetro puede actuar como entrada inversa.
- Si falta información, devuelve `values:{}` y una advertencia; nunca inventa un resultado.
- Devuelve siempre `{ values, warnings, formula }`.
- `formatNumber` usa hasta 12 cifras significativas y notación científica para magnitudes no nulas menores que `1e-6` o mayores o iguales que `1e9`.
- Las dimensiones de campo son 1, 2, 3 o 4 y gobiernan el sufijo de unidad y la conversión.

Categorías exactas:

```text
basic_2d     2D básicas
polygons     Polígonos regulares
solids       Sólidos 3D
hypersolids  Geometría 4D
converters   Conversores
others       Formas paramétricas
```

#### 4.1 Catálogo y fórmulas exactas

Usa estos identificadores y no otros. Conserva las definiciones indicadas.

1. `circle` - Círculo. Entradas inversibles: radio, diámetro, circunferencia o área. `d=2r`, `C=2*pi*r`, `A=pi*r^2`.
2. `square` - Cuadrado. Entradas inversibles: lado, diagonal, perímetro o área. `d=a*sqrt(2)`, `P=4a`, `A=a^2`.
3. `rectangle` - Rectángulo. Requiere ancho y altura. `d=sqrt(w^2+h^2)`, `P=2(w+h)`, `A=wh`.
4. `triangle` - Triángulo equilátero. Entradas inversibles: lado, altura, perímetro o área. `h=a*sqrt(3)/2`, `P=3a`, `A=a^2*sqrt(3)/4`.
5. `oval` - Elipse. Los ejes mayor y menor son diámetros completos. Con semiejes `a=ejeMayor/2`, `b=ejeMenor/2`, `A=pi*a*b`; perímetro por Ramanujan II: `h=(a-b)^2/(a+b)^2`, `P=pi*(a+b)*(1+3h/(10+sqrt(4-3h)))`.
6. `rhombus` - Rombo. Requiere diagonales `p,q`. `lado=sqrt(p^2+q^2)/2`, `P=4*lado`, `A=pq/2`.
7. `parallelogram` - Paralelogramo. Requiere base, lado oblicuo y altura. `P=2(base+lado)`, `A=base*altura`. Advierte si altura > lado.
8. `trapezoid` - Trapecio. Requiere base mayor, base menor y altura. `mediana=(B+b)/2`, `A=(B+b)h/2`.
9. `semicircle` - Semicírculo. Entradas inversibles: radio, diámetro, longitud de arco, perímetro total o área. `arco=pi*r`, `P=(pi+2)r`, `A=pi*r^2/2`.
10. `pentagon` - Pentágono regular, `n=5`.
11. `hexagon` - Hexágono regular, `n=6`.
12. `heptagon` - Heptágono regular, `n=7`.
13. `octagon` - Octágono regular, `n=8`.
14. `nonagon` - Nonágono regular, `n=9`.
15. `decagon` - Decágono regular, `n=10`.

Para todo polígono regular: entradas inversibles lado, perímetro, apotema o área. `P=n*s`, `apotema=s/(2*tan(pi/n))`, `A=n*s^2/(4*tan(pi/n))`. Inversas: `s=P/n`, `s=2*apotema*tan(pi/n)`, `s=sqrt(4*A*tan(pi/n)/n)`.

16. `sphere` - Esfera. Entradas inversibles radio, diámetro, superficie o volumen. `S=4*pi*r^2`, `V=4*pi*r^3/3`.
17. `cube` - Cubo. Entradas inversibles lado, diagonal espacial, superficie o volumen. `d=a*sqrt(3)`, `S=6a^2`, `V=a^3`.
18. `cylinder` - Cilindro. Requiere radio y altura. `S=2*pi*r*(r+h)`, `V=pi*r^2*h`.
19. `cone` - Cono recto. Requiere radio y altura. `g=sqrt(r^2+h^2)`, `lateral=pi*r*g`, `S=pi*r*(r+g)`, `V=pi*r^2*h/3`.
20. `pyramid` - Pirámide cuadrada. Requiere lado de base y altura. `g=sqrt(h^2+(a/2)^2)`, `S=a^2+2ag`, `V=a^2*h/3`.
21. `prism` - Prisma rectangular. Requiere longitud, ancho y altura. `d=sqrt(l^2+w^2+h^2)`, `S=2(lw+lh+wh)`, `V=lwh`.
22. `tesseract` - Tesseract. Cualquier magnitud es entrada inversible: lado `a`, diagonal 4D `d4`, área total de 24 caras cuadradas `A2`, volumen total de la frontera cúbica `V3` o 4-volumen `V4`. Fórmulas: `d4=2a`, `A2=24a^2`, `V3=8a^3`, `V4=a^4`. Inversas: `a=d4/2`, `a=sqrt(A2/24)`, `a=cbrt(V3/8)`, `a=V4^(1/4)`.
23. `hypersphere` - 3-sphere (S³), frontera de una 4-ball B⁴. Entradas inversibles: radio, diámetro, volumen 3D de la frontera o 4-volumen interior. `V3(S3)=2*pi^2*r^3`, `V4(B4)=pi^2*r^4/2`. Inversas exactas correspondientes.
24. `pentachoron` - 4-simplex regular; también pentatope, pentachoron o 5-cell. Cualquier magnitud es inversible: lado, altura 4D, circunradio, inradio, volumen 3D total de las cinco celdas tetraédricas o 4-volumen. `h=a*sqrt(5/8)`, `R=a*sqrt(2/5)`, `r=a/sqrt(40)`, `V3=5*a^3/(6*sqrt(2))`, `V4=sqrt(5)*a^4/96`. Implementa todas sus inversas.
25. `length_converter` - Conversor de longitud, potencia 1.
26. `area_converter` - Conversor de área, potencia 2.
27. `volume_converter` - Conversor de volumen, potencia 3.
28. `star` - Estrella regular de cinco puntas como polígono canónico de 10 vértices alternos, radio exterior `R` e interior `r`. `arista=sqrt(R^2+r^2-2Rr*cos(pi/5))`, `P=10*arista`, `A=5Rr*sin(pi/5)`. Advierte si `r>=R`.
29. `heart` - Corazón paramétrico canónico `x=16*sin^3(t)`, escalado al ancho `w`. Entrada inversible por ancho o área. `A=45*pi*w^2/256`.
30. `arrow` - Flecha paramétrica de longitud `l` y ancho `w`. Cuerpo de longitud `0.6l` y ancho `0.4w`; punta triangular de longitud `0.4l`; `A=0.44lw`.
31. `cross` - Cruz simétrica como unión de dos rectángulos ortogonales iguales. Ancho de brazo `w`, longitud total `L`; `A=2Lw-w^2`. Advierte si `w>=L`.

No sustituyas estas definiciones por otras convenciones geométricas.

#### 4.2 Unidades exactas

Define factores lineales respecto del metro:

```js
{
  nm: 1e-9,
  'µm': 1e-6,
  mm: 1e-3,
  cm: 1e-2,
  m: 1,
  km: 1e3,
  in: 0.0254,
  ft: 0.3048,
  yd: 0.9144,
  mi: 1609.344,
  au: 149597870700,
  ly: 9.4607304725808e15,
  pc: 3.085677581491367e16
}
```

La conversión entre unidades debe elevar el cociente de factores a la dimensión: `value*(from/to)^power`. La UI muestra sufijos lineales, cuadrados, cúbicos o cuárticos. La unidad inicial es `cm`.

### 5. Analizador local de voz

Implementa `src/ai/local-parser.js` sin IA. Exporta `parseVoiceCommand(transcript)` y devuelve:

```js
{ shapeId, dimensions, requestedQuantities, transcript }
```

Debe reconocer en castellano y, cuando ya existen, equivalentes ingleses de:

- Todas las figuras 2D y 3D del catálogo.
- Tesseract: `tesseract`, `teseracto`, `tesseracto`, `hipercubo`.
- 3-sphere: `3-sphere`, `3 sphere`, `tres esfera`, `hiperesfera`, `glomo`.
- 4-simplex: `4-simplex`, `4 simplex`, `cuatro simplex`, `pentacoron`, `pentachoron`, `pentatope`, `5-cell`.
- Estrella, corazón, flecha y cruz.
- Campos: radio, diámetro, ancho, altura, lado/arista, base, longitud, área, volumen, perímetro, superficie, apotema, bases mayor/menor, ejes mayor/menor, diagonales mayor/menor, radios exterior/interior, ancho de brazo, longitud total, volumen frontera e hipervolumen.
- Unidades habladas: nanómetros, micrómetros, milímetros, centímetros, metros, kilómetros, pulgadas, pies, yardas y millas, además de abreviaturas contempladas.
- Verbos de petición: `calcula`, `averigua`, `dame`.

Acepta números negativos en el análisis, pero deja que el motor los rechace por dominio. Conserva texto original por dimensión en `rawText`, marca `source:'voice'` y confianza 1 para coincidencias locales.

### 6. Proceso principal, privacidad y seguridad

En `electron/main.cjs`:

- Crea `BrowserWindow` de 1500x950, mínimo 1080x720, fondo `#07111f`, título `Calculadora Geo`, menú oculto.
- `contextIsolation:true`, `nodeIntegration:false`, `sandbox:true`, `webSecurity:true`.
- Carga `src/index.html` como archivo local.
- Niega ventanas nuevas y navegación.
- Permite exclusivamente permisos `media`, `camera` y `microphone`.
- Toda petición de red de IA se realiza en el proceso principal. El renderer tiene `connect-src 'none'`.
- Estado en `%LOCALAPPDATA%\CalculadoraGeo\settings.json` mediante escritura temporal y renombrado atómico.
- Estado inicial: `{settings:{localStrict:true,language:'es'}}` y tres perfiles.
- Perfiles predeterminados:
  - Ollama local - `http://127.0.0.1:11434/v1`
  - LM Studio local - `http://127.0.0.1:1234/v1`
  - llama.cpp local - `http://127.0.0.1:8080/v1`
- Protocolo interno: `openai-chat`.
- En Local estricto solo acepta host `127.0.0.1`, `localhost`, `::1` o `[::1]`; rechaza lo demás con un mensaje claro.
- Fuera de Local estricto admite HTTP/HTTPS de proveedores cloud configurados manualmente.
- Cifra claves API con `safeStorage.encryptString`, que en Windows usa protección ligada al usuario; almacena `enc:<base64>`. Nunca expongas la clave al renderer: devuelve solo `hasApiKey`.
- No imprimas claves en logs.
- Usa `fetch` con `AbortController`; timeout general 120 s y listado de modelos 15 s.
- Resuelve modelo con el configurado o el primer modelo de `GET /models`.
- Chat mediante `POST /chat/completions`, cuerpo `{model,messages,temperature,max_tokens}`.
- Interpreta respuestas `choices[0].message.content`, contenido en array, `output_text` o `output[].content[].text`.
- Muestra el error real del proveedor cuando una capacidad no existe; no anuncies compatibilidad ficticia.

Registra exclusivamente estos IPC:

```text
state:get
settings:save
profile:save
profile:delete
profile:test
ai:explain
ai:recognize-image
ai:transcribe-audio
app:open-data-folder
```

`electron/preload.cjs` debe exponer con `contextBridge` solo `window.geoDesktop` con métodos equivalentes a esos IPC. No expongas `ipcRenderer`, Node, archivos ni shell de forma genérica.

### 7. Contratos de IA

#### 7.1 Explicación

El motor local calcula primero. La IA recibe figura, unidad, entradas, resultados y fórmula. El prompt le ordena explicar en castellano fórmula, sustitución y unidades, con temperatura 0.1, sin alterar ningún resultado determinista.

#### 7.2 Imagen y cámara

Admite exclusivamente PNG, JPEG/JPG y WebP como Data URL. Rechaza una Data URL mayor de 16.000.000 caracteres. Envía al modelo de visión la imagen junto a una lista limitada del catálogo. Temperatura 0 y máximo 1800 tokens.

El mensaje al modelo debe exigir:

- Identificar una o varias figuras.
- Asociar números, unidades, símbolos y texto manuscrito o impreso con la dimensión correcta.
- No inventar valores ausentes.
- No estimar medidas mediante píxeles.
- Usar solo `shapeId` del catálogo cuando exista coincidencia.
- Devolver exclusivamente JSON válido con este esquema:

```json
{
  "figures": [
    {
      "shapeId": "string|null",
      "shapeName": "string",
      "confidence": 0.0,
      "dimensions": [
        {
          "field": "string",
          "value": 0,
          "unit": "cm",
          "rawText": "texto",
          "confidence": 0.0
        }
      ],
      "warnings": ["..."]
    }
  ],
  "requestedQuantities": ["area"],
  "ambiguities": ["..."]
}
```

El parser de respuesta elimina cercas Markdown, intenta JSON completo y después el fragmento entre la primera `{` y la última `}`. Si sigue siendo inválido, informa del error.

La UI muestra figura, confianza, dimensiones, unidades, avisos y ambigüedades. El botón **Aplicar y calcular** permanece desactivado hasta existir una detección. Solo al pulsarlo se transfieren los datos a la calculadora. Convierte cada valor desde su unidad reconocida a la unidad actual respetando la dimensión del campo.

#### 7.3 Voz y audio

- Dictado local del sistema con `SpeechRecognition` o `webkitSpeechRecognition`, `es-ES`, resultados intermedios, no continuo.
- Si no está disponible, muestra la alternativa de grabar o subir audio.
- Grabación con `getUserMedia({audio:true})` y `MediaRecorder`.
- Archivo `audio/*` permitido.
- Transcripción mediante `POST /audio/transcriptions` con multipart: archivo, modelo y `language=es`.
- Límite binario: 25 MiB.
- Modelo: el del perfil o `whisper-1` si está vacío.
- Acepta `text` o `transcript` en la respuesta.
- Tras transcribir, pasa el texto al analizador local y exige revisión humana antes de aplicarlo.

### 8. Interfaz exacta

La aplicación está íntegramente en castellano y tiene estética oscura, técnica y limpia de Windows 11. No uses un menú nativo. Usa Segoe UI Variable/Segoe UI y Consolas para cifras y fórmulas.

Variables de color exactas:

```css
--bg:#07111f;
--panel:#0d1b2c;
--panel2:#102238;
--line:#203a55;
--text:#edf6ff;
--muted:#87a1b9;
--cyan:#27d5d5;
--cyan2:#0d9399;
--purple:#9b7bff;
--amber:#ffbe55;
--red:#ff6b79;
--green:#4ee0a1;
--shadow:0 18px 50px rgba(0,0,0,.24);
```

Fondo `radial-gradient(circle at 70% 0,#102a43 0,transparent 35%),#07111f`. Paneles con degradado oscuro, borde de 1 px, radio 16 px y sombra. Controles con radios 8-13 px. Acentos cian, geometría auxiliar púrpura, advertencias ámbar, errores rojos y estado local verde.

#### 8.1 Cabecera

- Altura 82 px, borde inferior.
- Marca cuadrada de 44 px con degradado cian-púrpura y texto `G⁴`.
- Título `Calculadora Geo` y subtítulo `Geometría exacta · Windows 11 · Local`.
- Tres pestañas: `Calculadora`, `Cámara, archivo y voz`, `IA y ajustes`.
- Insignia derecha `● Local estricto`; al desactivarlo, `● Cloud opcional`.

#### 8.2 Vista Calculadora

- Layout principal: columna lateral de 286 px y espacio de trabajo, separación 18 px, relleno 18 px.
- Lateral: buscador, chips de las seis categorías y lista filtrada. Iconos textuales: `4D`, `3D`, `↔` o `◇`.
- Hero: categoría, figura, descripción, selector de unidad y botón Limpiar.
- Zona central: datos conocidos editables, advertencias y tarjeta de relación matemática.
- Lado derecho: SVG simbólico normalizado y resultados.
- Tarjeta inferior `PROFESOR IA OPCIONAL`, selector de perfil, botón Explicar y respuesta.
- Los campos azules son editables. Cada cambio recalcula inmediatamente.
- Al cambiar de unidad, convierte entradas existentes según dimensión y recalcula.
- Copiar resultados produce el nombre de figura y líneas `Etiqueta: valor`.
- SVG local para círculo, esfera/3-sphere, cuadrado/cubo, triángulo/pirámide/4-simplex, tesseract, rectángulo/prisma, cilindro, cono, elipse, estrella, corazón, flecha y cruz; polígono genérico para el resto. No cargues imágenes externas.

Medidas de referencia: paneles radio 16 px; inputs 48 px y texto numérico 19 px Consolas; visual 205 px; la cuadrícula principal usa `minmax(480px,1.15fr)` y `minmax(360px,.85fr)`; por debajo de 1250 px se apila y se oculta la tarjeta visual.

#### 8.3 Vista Cámara, archivo y voz

- Encabezado `Reconocer y calcular`, explicación y selector de perfil.
- Dos paneles: imagen/cámara con escenario de 360 px y voz/dictado con orbe de 112 px.
- Botones: Subir imagen, Abrir cámara, Capturar, Reconocer; Dictado del sistema, Grabar para IA, Audio, Interpretar.
- Mensaje visible: `PNG, JPEG o WebP · sin estimar medidas por píxeles`.
- Aviso de privacidad del perfil seleccionado.
- Panel inferior `REVISIÓN HUMANA` y botón `Aplicar y calcular`.

#### 8.4 Vista IA y ajustes

- Encabezado `IA y privacidad` y botón `Abrir carpeta de datos`.
- Tres paneles: modo de red, lista de perfiles y editor.
- Interruptor Local estricto.
- Aviso `Sin telemetría · sin login · sin actualizador · sin puertos LAN`.
- Editor: nombre, URL base, modelo y clave API; Guardar, Probar conexión y Eliminar.
- Dejar la clave vacía al editar conserva la cifrada existente.
- Toast inferior derecho, visible 3,5 s.

### 9. HTML y CSP

`src/index.html` tendrá `lang="es"`, `UTF-8`, viewport y esta CSP exacta:

```html
default-src 'self'; img-src 'self' data: blob:; media-src 'self' blob:; script-src 'self'; style-src 'self' 'unsafe-inline'; connect-src 'none';
```

No uses scripts inline. Carga `styles.css` y `app.js` como módulo. Añade etiquetas, estados accesibles, `aria-live` para advertencias y toast, y `playsinline` en vídeo.

### 10. Estado y comportamiento del renderer

Estado inicial exacto:

```js
{
  shapeId:'circle',
  category:'basic_2d',
  unit:'cm',
  inputs:{},
  result:{values:{},warnings:[],formula:''},
  appState:{settings:{localStrict:true},profiles:[]},
  imageDataUrl:null,
  recognition:null,
  stream:null,
  recorder:null,
  audioChunks:[]
}
```

Escapa HTML generado desde datos. No uses `innerHTML` con contenido del proveedor sin escape. Desactiva botones cuando falten imagen, perfil o reconocimiento. Detén pistas de cámara y micrófono al finalizar. No conserva imágenes ni grabaciones automáticamente.

### 11. Pruebas obligatorias

Crea `tests/math.test.mjs`. Deben resultar **38/38 pruebas superadas**:

- Una prueba de que el catálogo contiene exactamente 31 identificadores únicos.
- Una prueba parametrizada independiente para cada una de las 31 utilidades:
  - círculo r=2, área=4*pi;
  - cuadrado a=2, área=4;
  - rectángulo 3x4, diagonal=5;
  - triángulo a=2, área=sqrt(3);
  - elipse ejes 10 y 6, área=15*pi;
  - rombo diagonales 8 y 6, lado=5;
  - paralelogramo 5,4,3, área=15;
  - trapecio 8,4,3, área=18;
  - semicírculo r=2, área=2*pi;
  - polígonos regulares de lado 2, perímetros 10,12,14,16,18,20;
  - esfera r=2, volumen=32*pi/3;
  - cubo a=2, volumen=8;
  - cilindro r=2,h=3, volumen=12*pi;
  - cono r=3,h=4, generatriz=5;
  - pirámide a=3,h=4, volumen=12;
  - prisma 2x3x4, volumen=24;
  - tesseract a=2, 4-volumen=16;
  - 3-sphere r=2, 4-volumen=8*pi^2;
  - 4-simplex a=2, 4-volumen=sqrt(5)/6;
  - conversores desde 1 m: 100 cm, 10000 cm² y 1000000 cm³;
  - estrella R=5,r=2, área=50*sin(pi/5);
  - corazón ancho 10, área=45*pi*100/256;
  - flecha 10x4, área=17.6;
  - cruz w=2,L=6, área=20.
- Inversa de tesseract desde V4=81 produce lado 3.
- Inversa de 3-sphere desde V3=`2*pi^2*27` produce radio 3.
- Inversa de 4-simplex desde su V4 para lado 3 recupera lado 3.
- Conversión dimensional de 1 m a cm en potencias 1,2,3,4 produce 100, 10000, 1000000 y 100000000.
- Dictado `Calcula el volumen de un cilindro de radio 4 centímetros y altura 12 centímetros` identifica cilindro, radio 4, altura 12 y cm.
- Un cilindro con solo radio no devuelve valores y sí advertencias.

Usa comparación relativa con tolerancia `1e-9`, o `1e-8` en casos parametrizados.

### 12. Documentación

`README.md` y `ENTREGA.md` deben explicar en castellano:

- Inicio portable o mediante instalador.
- Que ambos binarios no están firmados y SmartScreen puede advertir.
- Que el cálculo funciona sin IA.
- Cómo configurar los tres runtimes locales.
- Que OpenAI-compatible no garantiza visión o audio en todos los modelos.
- Datos en `%LOCALAPPDATA%\CalculadoraGeo` y claves cifradas.
- Ausencia de telemetría, login, Store, actualizador y listener LAN.
- Comandos de desarrollo y compilación.

No afirmes que todos los modelos soportan todas las capacidades. La compatibilidad universal significa que cualquier runtime o nube que exponga los endpoints OpenAI-compatible usados puede configurarse; cada modelo debe aportar visión, chat o transcripción según la operación.

### 13. Construcción y validación final

Ejecuta en orden:

```powershell
npm install
npm test
npm run pack
npm run dist
npm run portable
```

Después:

1. Confirma 38/38 pruebas.
2. Arranca la app en desarrollo y comprueba que no hay errores de consola.
3. Arranca el directorio empaquetado y comprueba visualmente las tres vistas.
4. Ejecuta el portable sin instalación.
5. Verifica cámara/archivo, revisión de reconocimiento, dictado local y validación de perfiles; si no hay modelo externo disponible, valida los estados de error sin fingir éxito.
6. Comprueba con Authenticode que portable e instalador muestran `NotSigned`.
7. Calcula SHA-256 y anótalos en la entrega; no intentes forzar hashes históricos, pues cada compilación puede variar.
8. Copia binarios, fuente y documentación a `F:\HERRAMIENTAS\CALCULADORA-GEO`.

### 14. Definición de terminado

No declares el trabajo terminado hasta que se cumpla todo:

- Existen los dos `.exe` x64 con los nombres exactos.
- No están firmados.
- La aplicación abre en Windows 11 y las tres vistas son utilizables.
- El catálogo tiene exactamente 31 utilidades.
- Las 38 pruebas pasan.
- Tesseract, 3-sphere y 4-simplex calculan directa e inversamente.
- Las unidades respetan potencia 1-4.
- Los datos incompletos nunca generan resultados falsos.
- Imagen, cámara, voz y audio desembocan en revisión humana antes del cálculo.
- Local estricto bloquea cualquier host no loopback.
- Claves protegidas y nunca enviadas al renderer.
- No hay telemetría, cuenta, updater, Store ni listener LAN.
- La apariencia y textos coinciden con este contrato.
- El código fuente completo se entrega sin secretos, sin placeholders y listo para GitHub.

Al finalizar, responde con un informe breve que enumere archivos creados, pruebas, binarios, estado de firma, hashes SHA-256 y cualquier capacidad que dependa de un modelo externo.

## FIN DEL PROMPT

---

Este prompt define la reconstrucción funcional y visual de **Calculadora Geo 1.0**. Los hashes de una nueva compilación no tienen por qué coincidir con otra compilación, aunque el código y el comportamiento sí coincidan.
