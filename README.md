# Protocolo I2S (Inter-IC Sound)

Es un protocolo de bus serie sincronico, que se diseño especificamente para transmitir datos de **audio digital**.

Este protocolo usa principalmente tres lineas de señal **BCLOCK, RLCLOK Y SD** sin embargo tambien puede llegar a usar una cuarta linea de señal **MCLOCK**.

---
## BCLOK

Esta señal son impulsos los cuales permiten sincronisar la informacion y evitar desfaces que dañen el audio, durante la subida del impulso se lee el bit de informacion que se enviara y en la bajada se carga, de esta forma se permite que el bit de informacion se envie de forma correcta.


Su frecuencia depende de la frecuencia de muestreo, la resolución en bits y la cantidad de canales:
$$\text{Frecuencia BCLK} = F_{\text{muestreo}} \times N_{\text{bits}} \times N_{\text{canales}}$$
---
## LRCLOK

Determina a qué canal de audio corresponde el dato que se está transmitiendo. Su frecuencia es exactamente igual a la frecuencia de muestreo ($F_s$).
* **LRCLK = 0 (Nivel Bajo):** Canal Izquierdo (*Left Channel*).
* **LRCLK = 1 (Nivel Alto):** Canal Derecho (*Right Channel*).

Ademas el BCLOK tiene un ciclo de desface, ciclo en el cual ocurre el cambio de canal en el LRCLOK.

## SD 

Es la informacion del audio **PCM (Modulación por Código de Pulsos)** que se envia bit a bit a un **DAC** el cual se encarga de convertir esta informacion digital a una analoga en forma de sonido.

## MCLOK

Es una señal de reloj de alta frecuencia opcional pero requerida por muchos conversores (ADC/DAC) y DSPs para alimentar sus módulos de sobremuestreo (*oversampling*) y filtros digitales internos. Típicamente su frecuencia es $256 \times F_s$ (aunque también puede ser $128 \times F_s$, $384 \times F_s$ o $512 \times F_s$).

<p align="center">
  <img src="Imagenes/i2s.gif" alt="Animación I2S">
</p>

### Archivo .WAV

Un archivo **WAV** (Waveform Audio File Format) es un formato estándar de almacenamiento de audio digital sin compresión, desarrollado por Microsoft e IBM. Al no estar comprimido (generalmente utiliza codificación PCM), conserva la señal de audio original sin pérdida de calidad. Esto lo hace ideal para el procesamiento de audio en hardware y microcontroladores, aunque requiere más espacio de almacenamiento en comparación con formatos comprimidos como el MP3.

#### Estructura del archivo: Metadatos y Datos de Audio

Internamente, un archivo WAV es muy simple y se divide en dos bloques principales:

1. **Metadatos (Cabecera / Header):** Son los primeros bytes del archivo (normalmente los primeros 44 bytes). Contienen la "receta" del audio para que el sistema sepa cómo reproducirlo. Aquí se define la frecuencia de muestreo (*sample rate*, ej. 44100 Hz), la profundidad de bits (*bit depth*, ej. 16 bits), el número de canales (mono o estéreo) y el tamaño del archivo.

*   **Lectura de los Metadatos (Estructura RIFF/WAVE):** La cabecera se rige por el estándar RIFF. La mayoría de los valores numéricos se guardan en formato **Little-Endian** (el byte menos significativo va primero). Para leerlos, el sistema carga los primeros 44 bytes y busca palabras en código ASCII (como `RIFF` y `WAVE`) para validar el archivo. Luego, extrae los parámetros matemáticos de posiciones exactas.

2. **Datos de Audio (Payload):** Es el audio puro. Se trata de una secuencia inmensa de números (muestras PCM) ubicada inmediatamente después de la cabecera. Es la representación matemática de la onda de sonido original.

### El Estándar RIFF (Resource Interchange File Format)

**RIFF** es un formato "contenedor" o plantilla universal desarrollado por Microsoft e IBM en 1991 para estructurar archivos multimedia. Define cómo se deben organizar los datos dentro del archivo dividiéndolos en bloques estandarizados llamados **Chunks** (fragmentos).

#### Estructura de un "Chunk"

Todo archivo basado en el estándar RIFF se construye apilando estos bloques. Cada *chunk* tiene siempre una estructura rígida dividida en tres partes fundamentales:

| Elemento | Tamaño | Descripción |
|---|---|---|
| **Identificador (FourCC)** | 4 bytes | Código de 4 caracteres ASCII que indica el tipo de bloque (ej. `RIFF`, `fmt `, `data`). |
| **Tamaño del Chunk** | 4 bytes | Número entero (almacenado en formato Little-Endian) que indica cuántos bytes ocupan los datos del bloque. |
| **Datos (Payload)** | Variable | La información real del bloque (configuraciones, valores matemáticos, o audio puro). |

> **Ventaja crítica para el hardware (FPGA/Microcontroladores):** Esta estructura modular es ideal para sistemas integrados. Si el código VHDL o C encuentra un *chunk* que no le interesa procesar (como etiquetas de copyright o portadas de álbumes), simplemente lee el campo "Tamaño del Chunk" y salta esa cantidad exacta de bytes hacia adelante, ignorando la información irrelevante sin corromper la lectura.

#### ¿Cómo se aplica el estándar RIFF a un archivo .WAV?

Un archivo WAV es, en esencia, un contenedor RIFF configurado específicamente para almacenar audio. Su estructura interna sigue esta jerarquía obligatoria:

1. **Bloque Padre (`RIFF`):** Es el contenedor general de todo el archivo. Su identificador es `RIFF`, e inmediatamente después de indicar su tamaño, incluye la "palabra mágica" `WAVE` para especificar qué tipo de recurso contiene.
2. **Sub-bloque de Formato (`fmt `):** Es el "mapa" del audio. Contiene los parámetros necesarios para configurar los relojes del bus I2S: codificación (generalmente PCM), cantidad de canales, frecuencia de muestreo (*sample rate*) y profundidad de bits (*bit depth*).
3. **Sub-bloque de Datos (`data`):** Es el cuerpo principal del archivo. Contiene la secuencia masiva e ininterrumpida de muestras de audio analógico cuantizadas (la señal PCM) que se inyectarán directamente en el flujo del bus I2S hacia el DAC.


#### Funcionamiento respecto al protocolo I2S

La relación entre el archivo WAV y el protocolo I2S en un proyecto funciona así:

1. **Lectura y Configuración:** Primero se leen los **metadatos** del archivo WAV almacenado. Con esta información (frecuencia de muestreo, canales y bits), se ajusta la velocidad de los relojes de su periférico I2S para que coincidan exactamente con el formato del archivo.
2. **Extracción y Envío:** El sistema descarta la cabecera (ya no la necesita) y comienza a leer los **datos de audio** en crudo. Estos datos se envían bit a bit a través de los pines del bus I2S hacia el amplificador.
3. **Reproducción:** El amplificador o DAC recibe el flujo I2S en tiempo real y lo convierte en las señales eléctricas analógicas que finalmente mueven el altavoz para generar el sonido.


