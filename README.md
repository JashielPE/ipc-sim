# Simulador de cluster (IPC sim)

Cluster sintético que se muestra en una tablet para desarrollar y validar un pipeline
de visión por computadora sin acceso a hardware real. Incluye marcadores ArUco para
normalización geométrica y una tira de ground truth que codifica el estado en la
misma imagen que se analiza, de modo que el etiquetado de capturas es automático.

Además genera, de forma controlada y etiquetada, las condiciones adversas que una
detección robusta tiene que soportar: brillo del tablero sin ajustar, reflejos sobre
la pantalla y desplazamiento del contenido respecto a los ROIs.

## Alcance y confidencialidad

**Este simulador reproduce características genéricas de un cluster, nunca la identidad
de un producto real.** Los telltales son iconos genéricos tipo ISO 2575, las escalas
son arbitrarias, los textos de aviso son términos comunes de la industria y la
distribución es propia.

No debe incorporar logotipos, tipografías, distribución exacta ni textos literales de
ningún cliente, y este repositorio no debe contener imágenes de un cluster real. Si
alguna vez hace falta una versión que sí se parezca a un producto concreto, esa vive
en un repositorio privado y se sirve desde la red local.

Antes de publicar cualquier cosa derivada de trabajo profesional conviene revisar las
cláusulas de confidencialidad y propiedad intelectual del contrato laboral y
consultarlo con el jefe directo o con el área legal.

## Instalar en la tablet

1. Abrir la URL de GitHub Pages en Chrome.
2. Menú de tres puntos, "Agregar a la pantalla principal" o "Instalar app".
3. El icono abre el simulador en pantalla completa y en horizontal.

Queda cacheado, así que arranca sin conexión. Para recibir una actualización necesita
conexión al menos una vez.

Antes de capturar: brillo de la tablet al máximo y **brillo automático del sistema
desactivado**, para que la única variación de luminancia sea la del simulador, que sí
queda etiquetada.

## Controles

| Tecla | Acción | Tecla | Acción |
|---|---|---|---|
| `m` | MIL ámbar | `s` | Escenario de maduración |
| `w` | Triángulo rojo | `a` | Ciclo de avisos |
| `b` | Freno rojo | `c` | Barrido de agujas |
| `j` | Luz alta | `d` | Generar dataset |
| `l` / `r` | Direccionales | `x` | Reiniciar |
| `p` | Brillo parcial | `o` | Siguiente desplazamiento |
| `t` | Aviso visible | `e` | Reflejo simulado |
| `n` | Barra roja inferior | `h` | Panel de control |
| `v` | Variante V2 | `g` / `k` | Tira / marcadores |

Flechas arriba y abajo cambian la velocidad; izquierda y derecha, las rpm. Con Shift
el paso es mayor. Doble toque en el fondo muestra u oculta el panel.

Parámetros de URL:

```
index.html?speed=120&rpm=3000&dim=65&glare=1&off=5&notice=1&warning=2&redbar=1
```

Y `window.cluster` expone `setState`, `resetAll`, `runScenario`, `runCycle`,
`runDataset`, `getState` y `getLog`.

## Geometría canónica

El escenario mide siempre 1280 × 800. Centros de los marcadores ArUco del diccionario
`DICT_4X4_50`:

| ID | Posición | Centro |
|---|---|---|
| 0 | superior izquierda | (56, 56) |
| 1 | superior derecha | (1224, 56) |
| 2 | inferior derecha | (1224, 744) |
| 3 | inferior izquierda | (56, 744) |

Esos cuatro puntos son el destino de la homografía. Regiones en el espacio canónico,
**con desplazamiento cero**:

| Región | Origen | Tamaño |
|---|---|---|
| Pantalla (marco) | (60, 140) | 1160 × 470 |
| Fila de telltales | (240, 172) | 800 × 56 |
| Tacómetro | (130, 262) | 260 × 260 |
| Lectura central | (460, 268) | 360 × 230 |
| Velocímetro | (890, 262) | 260 × 260 |
| Aviso (icono y texto) | (420, 262) | 440 × 240 |
| Barra roja inferior | (60, 570) | 1160 × 24 |
| Odómetro | (120, 540) | texto 18 px |
| Indicador de marcha | (600, 516) | texto 24 px |
| Temperatura exterior | (1000, 540) | texto 18 px |
| Tira de ground truth | (120, 630) | 40 celdas de 26 × 44 |

Dentro de la fila de telltales el orden es: freno, triángulo, MIL, luz alta,
direccional izquierda, direccional derecha. Iconos de 42 px con separación de 30 px.

Escalas: velocímetro 0 a 240 km/h, tacómetro 0 a 6000 rpm. Ambas barren 240 grados,
de -120 a +120 respecto a la vertical.

Los textos pequeños (odómetro, marcha, temperatura) existen a propósito: dan objetivos
de 18 px para practicar lectura de texto chico, que es donde el template matching y el
OCR sufren.

## Tira de ground truth

40 celdas, blanco es 1 y negro es 0. La celda 0 siempre es blanca y la 39 siempre
negra: son las referencias para umbralizar sin suponer niveles absolutos. La tira y
los marcadores nunca se atenúan, nunca reciben reflejo y nunca se desplazan.

| Celda | Contenido |
|---|---|
| 0 | Referencia blanca |
| 1 | Estímulo disparado (T0 del escenario) |
| 2 | MIL |
| 3 | Triángulo rojo |
| 4 | Freno rojo |
| 5 | Luz alta |
| 6 | Direccional izquierda |
| 7 | Direccional derecha |
| 8 | Aviso visible |
| 9 | Variante V2 (recuadro) |
| 10 | Barra roja inferior |
| 11 | Telltales a brillo parcial |
| 12 | Reflejo simulado activo |
| 13–19 | Velocidad ÷ 2 (7 bits, MSB primero) |
| 20–26 | RPM ÷ 50 (7 bits, MSB primero) |
| 27–33 | Brillo del tablero, 0 a 100 (7 bits, MSB primero) |
| 34–37 | Índice de desplazamiento, 0 a 15 (4 bits, MSB primero) |
| 38 | Paridad par sobre las celdas 1 a 37 |
| 39 | Referencia negra |

Las divisiones son exactas: 240 ÷ 2 y 6000 ÷ 50 dan 120, dentro de los 127 que caben
en siete bits.

El índice de desplazamiento mapea a estos pares (dx, dy) en píxeles canónicos:

```
0:(0,0)    1:(-14,-9) 2:(-14,0) 3:(-14,9) 4:(-7,-9) 5:(-7,9) 6:(0,-9) 7:(0,9)
8:(7,-9)   9:(7,9)   10:(14,-9) 11:(14,0) 12:(14,9) 13:(-7,0) 14:(7,0) 15:(0,14)
```

Para cada captura puedes recuperar la posición real de un ROI sumando ese par a las
coordenadas de la tabla de geometría.

## Condiciones adversas

**Brillo del tablero** (0 a 100): se aplica como filtro sobre la pantalla y su
contenido. Mapea a un rango de luminancia de 0.25 a 1.0.

**Reflejo simulado** (`e`): gradiente blanco concentrado en las orillas y una banda
diagonal, imitando el comportamiento de un vidrio curvo donde las orillas reflejan y
el centro se mantiene limpio. Es un complemento controlado y etiquetado, no un
sustituto de capturar con la luz del cuarto encendida: el reflejo real tiene
estructura, color y bordes duros que un gradiente no reproduce. Conviene tener ambas
fuentes de variación en el dataset.

**Desplazamiento del ROI** (`o` o el deslizador): mueve el contenido de la pantalla
sin mover los marcadores ni la tira. Simula un cluster recolocado o un ajuste de
montaje, que es el caso donde una detección con ROI rígido falla aunque la
normalización por homografía sea correcta.

## Modos automáticos

**Escenario** (`s`): levanta el bit de estímulo y, tras el retardo configurado (5000 ms
por omisión), enciende el MIL, el aviso y la barra roja. Sirve para medir latencia
contra un valor conocido.

**Ciclo de avisos** (`a`): recorre la lista de avisos mostrando cada uno 3000 ms y
ocultándolo 1800 ms. Reproduce el patrón de indicaciones que aparecen y desaparecen.

**Barrido de agujas** (`c`): 12 posiciones de 1500 ms de cero a fondo de escala, para
construir la curva ángulo contra valor.

**Dataset** (`d`): 60 estados, tres niveles de brillo por dos estados de reflejo por
diez combinaciones de telltales y avisos, con el desplazamiento rotando entre estados.
La permanencia por estado es configurable.

El botón de descargar bitácora entrega un JSON con cada cambio de estado y su
timestamp, útil para verificar de forma cruzada lo que decodifique el lector de la
tira.

## Limitación importante

Un modelo entrenado contra esta tablet no transfiere a un cluster real: el panel, la
gamma, los subpíxeles, la curvatura del vidrio y los reflejos son distintos. Lo que
transfiere es el pipeline (captura, etiquetado, extracción de características,
entrenamiento, métricas) y la metodología de generar variación etiquetada. Los pesos y
los umbrales se vuelven a obtener con datos del banco.
