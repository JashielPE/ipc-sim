# Simulador de cluster (IPC sim)

Cluster sintético que se muestra en una tablet para desarrollar y validar un pipeline
de visión por computadora sin acceso a hardware real. Incluye marcadores ArUco para
normalización geométrica y una tira de ground truth que codifica el estado en la
misma imagen que se analiza, de modo que el etiquetado de capturas es automático.

## Alcance y confidencialidad

**Este simulador es genérico y no debe imitar ningún producto real.** Los iconos son
telltales estándar (ISO 2575), las escalas son arbitrarias y los textos son términos
comunes de la industria. El repositorio es público, así que cualquier ajuste de
tipografía, disposición o textos que acerque la apariencia a un cluster de cliente
debe vivir en un repositorio privado y servirse desde la red local.

## Instalar en la tablet

1. Abrir la URL de GitHub Pages en Chrome.
2. Menú de tres puntos, "Agregar a la pantalla principal".
3. El icono abre el simulador en pantalla completa, sin barra de direcciones.

Queda cacheado, así que arranca sin conexión. Para recibir una actualización necesita
conexión al menos una vez.

## Controles

| Tecla | Acción | Tecla | Acción |
|---|---|---|---|
| `m` | MIL ámbar | `s` | Correr escenario de maduración |
| `w` | Triángulo rojo | `c` | Barrido de agujas |
| `b` | Freno rojo | `d` | Generar dataset |
| `j` | Luz alta | `x` | Reiniciar |
| `l` / `r` | Direccionales | `h` | Panel de control |
| `p` | Brillo parcial | `g` | Tira de ground truth |
| `t` | Mensaje | `k` | Marcadores ArUco |
| `v` | Variante V2 | `f` | Pantalla completa |

Flechas arriba y abajo cambian la velocidad; izquierda y derecha, las rpm. Con Shift
el paso es mayor. Doble toque en el fondo muestra u oculta el panel.

También acepta parámetros de URL:

```
index.html?speed=120&rpm=3000&mil=1&dim=70&msg=MOTOR%20FAULT&variant=2
```

Y expone `window.cluster` con `setState`, `resetAll`, `runScenario`, `runDataset`,
`getState` y `getLog` para control desde la consola.

## Geometría canónica

El escenario mide siempre 1280 × 800 sin importar la pantalla. Centros de los
marcadores ArUco del diccionario `DICT_4X4_50`:

| ID | Posición | Centro |
|---|---|---|
| 0 | superior izquierda | (56, 56) |
| 1 | superior derecha | (1224, 56) |
| 2 | inferior derecha | (1224, 744) |
| 3 | inferior izquierda | (56, 744) |

Esos cuatro puntos son el destino de la homografía. Regiones de interés en el espacio
canónico resultante:

| Región | Origen | Tamaño |
|---|---|---|
| Fila de telltales | (120, 132) | 1040 × 66 |
| Tacómetro | (140, 230) | 300 × 300 |
| Panel central | (480, 238) | 320 × 290 |
| Velocímetro | (840, 230) | 300 × 300 |
| Tira de ground truth | (130, 580) | 34 celdas de 30 × 44 |

Escalas: velocímetro 0 a 240 km/h, tacómetro 0 a 6000 rpm. Ambas carátulas barren
240 grados, de -120 a +120 respecto a la vertical.

## Tira de ground truth

34 celdas, blanco es 1 y negro es 0. La celda 0 siempre es blanca y la 33 siempre
negra: sirven como referencias para umbralizar sin suponer niveles absolutos.

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
| 8 | Mensaje visible |
| 9 | Variante V2 |
| 10–16 | Velocidad ÷ 2 (7 bits, MSB primero) |
| 17–23 | RPM ÷ 50 (7 bits, MSB primero) |
| 24–30 | Brillo del tablero, 0 a 100 (7 bits, MSB primero) |
| 31 | Telltales a brillo parcial |
| 32 | Paridad par sobre las celdas 1 a 31 |
| 33 | Referencia negra |

Las divisiones son exactas: 240 ÷ 2 y 6000 ÷ 50 dan 120, dentro de los 127 que caben
en siete bits, así que la aguja recorre toda la carátula sin error de redondeo.

## Modos automáticos

**Escenario** (`s`): levanta el bit de estímulo y, tras el retardo configurado (5000 ms
por omisión), enciende el MIL y el mensaje en variante V2. Sirve para medir latencia
contra un valor conocido.

**Barrido de agujas** (`c`): 12 posiciones de 1500 ms recorriendo ambas carátulas de
cero a fondo de escala, para construir la curva ángulo contra valor.

**Dataset** (`d`): 24 estados, tres niveles de brillo por ocho combinaciones de
telltales, incluyendo brillo parcial como negativo difícil. La permanencia por estado
es configurable.

El botón de descargar bitácora entrega un JSON con cada cambio de estado y su
timestamp, útil para verificar de forma cruzada lo que decodifique el lector de la
tira.

## Limitación importante

Un modelo entrenado contra esta tablet no transfiere a un cluster real: el panel, la
gamma, los subpíxeles y los reflejos son distintos. Lo que transfiere es el pipeline
(captura, etiquetado, extracción de características, entrenamiento, métricas). Los
pesos y los umbrales se vuelven a obtener con datos del banco.
