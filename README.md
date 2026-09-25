# Laboratorio 3: Cálculo ambulatorio del Índice Pletismográfico Quirúrgico (SPI)  

## Integrantes
* Laura Valentina Velásquez Castiblanco (5600846)
* Carlos Felipe Moreno Guzmán (5600881)
* Juan Andrés Mateus Durán (5600787)

## Objetivos

### Objetivo General
Desarrollar un sistema de medición continua del índice pletismográfico quirúrgico (SPI) en condiciones ambulatorias.

### Objetivos Específicos
- Reconocer las características fundamentales de la onda de pulso a partir de las cuales se obtiene el SPI.
- Construir un sistema que calcule el SPI en tiempo real y bajo condiciones ambulatorias.
- Validar el funcionamiento del sistema desarrollado mediante un método que induzca una respuesta fisiológica similar a la que produce el dolor agudo.

> ### Parte A

La fotopletismografía (PPG) permite registrar de forma no invasiva las variaciones del volumen sanguíneo periférico [1]. De la onda de pulso se extraen características como el intervalo entre pulsos y la amplitud, a partir de las cuales se calcula el índice pletismográfico quirúrgico (SPI), el cual estima el balance entre la nocicepción y la analgesia durante la anestesia general [2].

El objetivo de la Parte A fue construir el sistema de adquisición de la onda de pulso, así como definir el diseño experimental para inducir una respuesta fisiológica similar a la del dolor agudo mediante la técnica Cold Pressor Test (CPT).

### **Circuito de acondicionamiento y sensor**

<img width="710" height="401" alt="image" src="https://github.com/user-attachments/assets/09b37a9d-5b0e-448d-9f68-7bbb1389e0bd" />

Se montó en protoboard el circuito presentado en la figura anterior, tomado de la guía de laboratorio, el cual consta de las siguientes etapas:

- Etapa de emisión: un transistor 2N3904 que controla el LED emisor del optoacoplador.
- Etapa de detección: el fototransistor, cuya señal pasa por un filtro pasa-altas pasivo que elimina el nivel DC.
- Etapa de amplificación y filtrado: un filtro pasa-bajas activo con LM358 y un segundo amplificador, con potenciómetros para ajustar el offset y la amplitud.

Como sensor se usó el optoacoplador, modificado para funcionar como sensor de reflectancia. El emisor y el detector se separaron y se colocaron lado a lado, de modo que la luz reflejada por el dedo llegara al detector. La salida del circuito se conectó a una entrada analógica de una ESP32 y se verificó la captura de las variaciones del volumen sanguíneo. Aunque el circuito se montó y se probó satisfactoriamente, la captura final se realizó con un módulo MAX30102 conectado a la ESP32, el cual integra en un solo encapsulado LEDs rojo e infrarrojo, un fotodetector y un ADC interno.

La comunicación entre el MAX30102 y la ESP32 se estableció mediante el protocolo I2C, utilizando los pines por defecto de la ESP32 (SDA = GPIO21, SCL = GPIO22). Para el manejo del sensor se empleó la librería SparkFun MAX3010x, configurada con una corriente de LED de 60 (en una escala de 0 a 255), una frecuencia de muestreo de 100 Hz, un ancho de pulso de 411 µs y un rango del ADC de 4096, utilizando únicamente el canal infrarrojo (modo Red+IR) para la adquisición de la señal PPG. Los datos se transmitieron por el puerto serial, en pares de tiempo (ms) y valor de intensidad infrarroja, para su posterior procesamiento en MATLAB.

El montaje final empleado para la adquisición de la señal se presenta en la siguiente imagen.

<img width="1600" height="1523" alt="image" src="https://github.com/user-attachments/assets/e2419c80-5ac7-4ee2-92b3-5eba488abffd" />

### **Cold Pressor Test (CPT)**

El CPT es una prueba de estrés que consiste en sumergir una parte del cuerpo, clásicamente la mano o el antebrazo, en agua helada durante un tiempo definido. El frío activa los nociceptores y termorreceptores cutáneos y provoca una respuesta simpática, la cual se traduce en vasoconstricción periférica, aumento de la frecuencia cardíaca y de la presión arterial, y una sensación de dolor que se incrementa con el tiempo [3]. Por esta razón, el CPT se emplea en investigación como estímulo doloroso controlado, reproducible y seguro. En el contexto del SPI, la activación simpática inducida por un estímulo nociceptivo reduce la amplitud del pulso por vasoconstricción y acorta el intervalo entre latidos; ambos cambios elevan el valor del SPI. Por eso se espera un aumento durante el CPT y un retorno hacia el valor basal durante la recuperación.

Para aplicarlo en el laboratorio, en lugar de la inmersión en agua helada se empleó una variante simple: se le pidió al profesor, quien actuó como sujeto de prueba, que sostuviera un trozo de hielo en la mano izquierda, opuesta a la que portaba el sensor. La captura tuvo una duración total de 2 minutos, distribuida en tres fases: fase basal (0-40 s), en reposo y sin estímulo; fase de CPT (40-80 s), durante la cual el sujeto sostiene el hielo; y fase de recuperación (80-120 s), en la que se retira el hielo y el sujeto vuelve al reposo.

> ### Parte B

### **1. Revisión literaria: Definición matemática del índice pletismográfico quirúrgico (SPI)**

#### **1.1. Fundamento fisiológico**

El SPI fue desarrollado por GE Healthcare y descrito por primera vez por Huiku et al. en 2007, bajo el nombre inicial de Surgical Stress Index (SSI) [1]. Su propósito es construir una medida continua y objetiva del balance entre la estimulación nociceptiva (dolor) y el efecto analgésico durante la anestesia general, a partir de una señal ya disponible en el quirófano: la onda de pulso obtenida por pulsioximetría [2]. Esto evita instrumentación adicional, pues reutiliza el sensor de SpO₂ que casi todo paciente anestesiado ya lleva puesto.

Un estímulo doloroso activa el sistema nervioso simpático, lo que produce dos efectos medibles en la periferia:

- Vasoconstricción periférica, que reduce el volumen de sangre que entra al tejido con cada latido y, en consecuencia, disminuye la amplitud de la onda PPG (PPGA).
- Aumento de la frecuencia cardíaca, que acorta el intervalo entre latidos (HBI).

La analgesia (por ejemplo, con opioides) tiende a atenuar ambos efectos. Por esta razón, el SPI combina PPGA y HBI: son dos indicadores fisiológicos distintos del mismo fenómeno, uno más ligado al tono vasomotor simpático (PPGA) y el otro más ligado al efecto de fármacos opioides sobre el nodo sinusal (HBI).

#### **1.2. Variables de entrada**

- PPGA (Photoplethysmographic Pulse Wave Amplitude): diferencia entre el valor máximo y el mínimo de la señal PPG en cada latido (la componente AC de la onda de pulso).
- HBI (Heart Beat Interval): intervalo de tiempo entre dos picos sistólicos consecutivos; equivale al recíproco de la frecuencia cardíaca instantánea.

#### **1.3. Normalización**

Dado que la amplitud absoluta de la PPG y la frecuencia cardíaca basal varían considerablemente entre personas, ninguna de las dos variables se emplea en su forma bruta. GE Healthcare aplica una transformación de histograma sobre una ventana móvil de valores recientes de PPGA y HBI: cada nueva muestra se reubica según el percentil que ocupa dentro de la distribución de valores anteriores del mismo paciente, generando así PPGAnorm y HBInorm, ambos acotados entre 0 y 100 [4]. Esto es lo que permite comparar el SPI entre pacientes distintos, en lugar de limitarse a comparar cambios relativos dentro de un mismo paciente.

Al iniciar la monitorización, el algoritmo requiere un período de "aprendizaje" (cercano a 3 minutos) para construir esa distribución de valores basales antes de que el número de SPI sea confiable; antes de ese punto, el valor se muestra en gris.

#### **1.4. Fórmula matemática e interpretación del SPI**

$$SPI = 100 - (0.7 \times PPGA_{norm} + 0.3 \times HBI_{norm})$$

Esta es la fórmula original reportada por Huiku et al. (2007) y confirmada en la documentación técnica de GE Healthcare [4]. El peso de 0,7 sobre PPGAnorm frente a 0,3 sobre HBInorm refleja que la amplitud del pulso responde de forma más marcada y más rápida al estímulo nociceptivo que el intervalo entre latidos.

El SPI es un número adimensional entre 0 y 100. Valores altos reflejan mayor actividad simpática o nocicepción (menor PPGAnorm y/o menor HBInorm), mientras que valores bajos reflejan analgesia adecuada o ausencia de estímulo doloroso. El rango de referencia para una anestesia bien balanceada en adultos sanos es 20-50, y se recomienda evitar incrementos súbitos mayores a 10 puntos, ya que estos son más indicativos de un evento nociceptivo agudo que el valor absoluto en sí [4].

Cabe señalar que el SPI, tal como lo definió GE Healthcare, está pensado para un paciente bajo anestesia general monitoreado con un pulsioxímetro clínico certificado, y emplea una ventana de normalización basada en varios minutos de datos del mismo sujeto. En este laboratorio se aplicó el mismo principio de cálculo (amplitud de pulso e intervalo entre latidos) a una señal PPG adquirida con un MAX30102 sobre la ESP32, en una persona consciente y en reposo, sin el algoritmo propietario de normalización histográfica del monitor comercial.

### **2. Captura y Procesamiento - C++ y MATLAB**

El procesamiento en tiempo real de la señal fotopletismográfica (PPG) proveniente del sensor MAX30102 se estructuró mediante una arquitectura distribuida entre el firmware del microcontrolador ESP32 (desarrollado en VS Code - PlatformIO) y el software de procesamiento numérico en MATLAB.

#### **2.1. Adquisición y Transmisión**

En el ESP32, se configuró la librería del sensor MAX30102 para muestrear el canal infrarrojo (IR) a una frecuencia de $100\text{ Hz}$ ($\Delta t = 10\text{ ms}$). El código gestiona el tiempo de muestreo mediante temporizadores por millis() para evitar bloqueos y envía la lectura limpia por la interfaz serie UART a $115200\text{ Baudios}$:

```cpp
#include <Arduino.h>
#include <Wire.h>
#include "MAX30105.h"

MAX30105 particleSensor;
unsigned long lastSampleTime = 0;
const unsigned long sampleInterval = 10; // 10 ms -> Fs = 100 Hz

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22); // Pines I2C SDA=21, SCL=22 en ESP32

  if (!particleSensor.begin(Wire, I2C_SPEED_FAST)) {
    while (1); // Error de inicialización
  }

  // Configuración del sensor MAX30102
  particleSensor.setup(0x1F, 4, 2, 100, 411, 4096); 
}

void loop() {
  if (millis() - lastSampleTime >= sampleInterval) {
    lastSampleTime = millis();
    uint32_t irValue = particleSensor.getIR(); // Canal IR para PPG
    
    // Envío de la lectura raw por puerto serie
    Serial.println(irValue); 
  }
}
```

#### **2.2. Acondicionamiento y Filtrado Digital**

La señal recibida por puerto serie en MATLAB se procesa mediante un filtro digital pasa-banda Butterworth de orden 3 ($0.7\text{ Hz} - 3.5\text{ Hz}$) utilizando la función filtfilt para lograr una respuesta de fase cero. Se invierte el signo para alinear las pulsaciones con picos positivos:

```matlab
% Filtrado pasa-banda Butterworth de orden 3
fs = 100;
[b, a] = butter(3, [0.7 3.5]/(fs/2), 'bandpass');
ppg_filtered = filtfilt(b, a, ppg_raw);

% Inversión de signo para orientación adecuada de picos sistólicos
ppg_filtered = -ppg_filtered;
```

#### **2.3. Detección de Picos y Valles - Método del Alpinista (MMDP)**

Para poder detectar satisfactoriamente los picos y valles de la señal PPG capturada, se implementó el método de detección denominado "Método del Alpinista". Este algoritmo identifica picos sistólicos evaluando el número de pasos ascendentes consecutivos ($num\_upsteps$) que cumplen $f(t_i) > f(t_{i-1})$. Para rechazar artefactos y evitar la falsa detección de la muesca dicrota, se requirió un umbral estricto de $8$ pasos ascendentes continuos y un intervalo de refractariedad de $0.35\text{ s}$ entre latidos:

```matlab
% Algoritmo MMPD: Detección de picos por pendientes ascendentes continuas
if (ppg_filtered(i) > ppg_filtered(i-1))
    num_upsteps = num_upsteps + 1;
else
    if (num_upsteps >= min_upsteps) && ((i - last_peak_idx) > min_hbi_samples)
        % Confirmación de pico sistólico
        peak_idx = i - 1;
        peaks = [peaks; peak_idx];
        
        % Localización del valle previo en el intervalo del latido
        [~, min_rel_idx] = min(ppg_filtered(last_peak_idx:peak_idx));
        valleys = [valleys; last_peak_idx + min_rel_idx - 1];
        
        last_peak_idx = peak_idx;
    end
    num_upsteps = 0;
end
```

#### **2.4. Extracción de Parámetros y Cálculo del SPI**

Por cada latido $k$ detectado, se calcula la amplitud pico a valle ($PPGA_k$) y el intervalo inter-latido ($HBI_k$). Ambas variables se normalizan en un rango relativo de 0 a 100 ($PPGA_{norm}$ y $HBI_{norm}$) y se computa el $SPI$:

```matlab
% Cálculo de parámetros por latido
PPGA = ppg_filtered(peaks(end)) - ppg_filtered(valleys(end));
HBI = (peaks(end) - peaks(end-1)) / fs;

% Normalización lineal relativa (0 - 100)
PPGA_norm = ((PPGA - min_PPGA) / (max_PPGA - min_PPGA)) * 100;
HBI_norm  = ((HBI  - min_HBI)  / (max_HBI  - min_HBI))  * 100;

% Ecuación del Índice Pletismográfico Quirúrgico
SPI = 100 - (0.7 * PPGA_norm + 0.3 * HBI_norm);
```

### **3. Evaluación del SPI bajo maniobra CPT**

Para evaluar la capacidad del sistema en la detección de respuestas simpáticas e inducción de vasoconstricción periférica, se ejecutó el protocolo Cold Pressor Test (CPT) durante $120\text{ segundos}$ continuos a un voluntario sano. Esta prueba se desarrolló mediante las siguientes etapas:

* **1. Fase de Reposo ($0 - 40\text{ s}$):** El sujeto permaneció en reposo hemodinámico y sin movimiento.
* **2. Estímulo Nociceptivo - CPT ($40 - 80\text{ s}$):** Agarre de bloque de hielo de la mano contralateral ($\approx 0 - 4\text{ }^\circ\text{C}$).
* **3. Fase de Recuperación ($80 - 120\text{ s}$):** Retiro del estímulo y retorno a condiciones de reposo.

La señal PPG capturada durante dos minutos se observa a continuación:

<img width="1414" height="912" alt="image" src="https://github.com/user-attachments/assets/da8c6d67-5c02-4b36-b0ed-2be113bee506" />

Durante los $120\text{ s}$ de registro, la señal fotopletismográfica (PPG) filtrada mostró una correcta identificación de picos y valles sistólicos mediante el algoritmo MMPD, evidenciando en los primeros $40\text{ s}$ una morfología estable con una amplitud pico a valle amplia de entre $+1500\text{ u.a.}$ y $-1500\text{ u.a.}$; sin embargo, al aplicar la prueba Cold Pressor Test ($40 - 80\text{ s}$), se observó una significativa reducción en la amplitud de la señal PPG debido a la vasoconstricción periférica (descendiendo los picos a $+600\text{ u.a.}$ y ascendiendo los valles a $-600\text{ u.a.}$), para posteriormente iniciar una reexpansión vascular progresiva durante la fase de recuperación ($80 - 120\text{ s}$) que retornó la amplitud a sus valores iniciales.

### **4. Evolución Temporal del SPI - Análisis Gráfico**

Para lograr una representación clara del comportamiento dinámico de la respuesta autonómica a lo largo del tiempo, se modificó el código original de MATLAB con el fin de almacenar las series temporales completas de las variables fisiológicas ($PPGA$ e $HBI$) y registrar los valores del $SPI$ calculados latido a latido. Esta modificación permitió generar un entorno gráfico centrado exclusivamente en la trayectoria temporal del índice ($SPI$ en función del tiempo $t$), facilitando la correlación directa entre las fases de la prueba y la variación cuantitativa del nivel de estrés o nocicepción del sujeto.

A continuación se presenta y analiza la gráfica resultante tras la ejecución del protocolo experimental _Cold Pressor Test_ (CPT):

<img width="1395" height="912" alt="image" src="https://github.com/user-attachments/assets/8fcfc0d2-c198-4bcb-9569-6100dda6b620" />

El trazado del $SPI$ evidencia con alta sensibilidad los tres estados hemodinámicos de la prueba: durante la línea base ($0 - 40\text{ s}$) el índice fluctúa de manera estable en una franja inactiva de bajo estrés entre $20$ y $40$ unidades; al iniciar el contacto de la mano contralateral con el hielo ($40 - 80\text{ s}$), se aprecia claramente cómo el $SPI$ sube de manera pronunciada hasta sostenerse en un rango elevado de $55\text{ a }67$ unidades por la vasoconstricción periférica y la descarga simpática ante el dolor térmico; finalmente, tras retirar la mano del estímulo ($80 - 120\text{ s}$), el $SPI$ vuelve a bajar progresivamente hasta estabilizarse nuevamente en sus niveles basales de reposo ($20 - 30$ unidades), registrando únicamente un pico transitorio puntual de $100$ unidades cerca del segundo $86$ debido a un artefacto.

> ### Parte C

### 1. Procedimiento General

El sistema de adquisición se construyó a partir de un sensor óptico **MAX30102**, que integra en un solo encapsulado un LED emisor y un fotodetector, conectado a una placa **ESP32** mediante comunicación con el módulo interno del sensor. El dedo del sujeto de prueba se colocó sobre el sensor para registrar, por reflectancia, las variaciones del volumen sanguíneo periférico (señal PPG), mientras la ESP32 transmitió los datos crudos (tiempo y valor IR) por puerto serial hacia MATLAB a una frecuencia de muestreo de **100 Hz**.

La captura se realizó siguiendo el protocolo completo del **Cold Pressor Test (CPT)**, con una duración total de 2 minutos dividida en tres fases: reposo inicial (0–40 s), aplicación de la maniobra CPT (40–80 s) y recuperación (80–120 s). Para inducir el estímulo doloroso, el sujeto de prueba sostuvo un trozo de hielo con la mano contraria a la que tenía el dedo apoyado sobre el sensor, de manera que la maniobra generara una respuesta simpática sistémica sin interferir directamente con la señal PPG adquirida.

Una vez finalizada la captura, la señal se procesó en MATLAB descartando los primeros 1.5 s por transitorio de estabilización y aplicando un filtro pasa banda Butterworth (0.7–3.5 Hz, orden 3) para aislar el componente cardiaco de la señal. Sobre la señal filtrada se aplicó el **método del alpinista** para la detección de picos y valles, a partir de los cuales se calcularon, latido a latido, la **amplitud pico-valle (PPGA)** y el **intervalo entre latidos (HBI)**; ambas variables se normalizaron entre 0 y 100 y se combinaron mediante la fórmula $SPI = 100 - (0.7 \times PPGA_{norm} + 0.3 \times HBI_{norm})$ para obtener el índice pletismográfico quirúrgico (SPI) en función del tiempo.

### 2. Resultados Obtenidos

La señal PPG filtrada permitió una detección clara y continua de picos y valles a lo largo de todo el registro, sin pérdida aparente de latidos ni saturación de la señal, lo que indica una adecuada calidad de contacto entre el dedo y el sensor durante los 2 minutos de captura. La amplitud pico-valle se mantuvo relativamente estable durante los primeros 20 s, con oscilaciones más marcadas hacia el final del reposo y durante la maniobra, coherentes con los cambios de tono vasomotor esperados.

En cuanto al SPI por latido, durante el reposo inicial (0–40 s) el índice osciló mayormente entre 20 y 40, con una caída puntual hasta valores cercanos a 12–18 alrededor de los 13–17 s. A partir de los 20 s se observó un ascenso progresivo que continuó durante la maniobra CPT (40–80 s), alcanzando una meseta sostenida entre 50 y 65. El valor más alto de todo el registro corresponde a un pico de **100** hacia el final de la maniobra y el inicio de la recuperación (≈83–87 s), seguido de un descenso gradual hasta valores de 25–32 al cierre de los 120 s, sin llegar a estabilizarse por completo en ese lapso.

### 3. Análisis de Resultados

**Análisis 1: Comparación del SPI obtenido con los valores de referencia usados en cirugía**

Los valores de SPI registrados durante el reposo inicial se ubicaron mayormente dentro del rango de 20–50 recomendado para una analgesia intraoperatoria adecuada, mientras que el ascenso observado durante la maniobra CPT y su transición hacia la recuperación llevó al índice por encima de ese rango, hasta el valor máximo de 100. Este comportamiento es consistente con lo esperado para un estímulo doloroso aplicado sin analgesia farmacológica: durante el CPT, el frío activa nociceptores y termorreceptores cutáneos que desencadenan **vasoconstricción periférica** y **taquicardia relativa**, ambos cambios que elevan el SPI. El sistema logró seguir este aumento de activación simpática durante la maniobra y su posterior recuperación, lo que confirma que el circuito y el algoritmo implementados responden adecuadamente a cambios de origen autonómico, aun cuando los valores absolutos no sean directamente comparables con los de un paciente anestesiado, debido a que el sujeto de prueba estaba consciente y mantenía un tono simpático basal propio de la vigilia, además de la actividad muscular necesaria para sostener el hielo.

> [!NOTE]
> El ascenso del SPI durante el CPT se explica principalmente por la **reducción de la amplitud del pulso (PPGA)**, efecto directo de la vasoconstricción periférica inducida por el frío; el acortamiento del intervalo entre latidos (HBI) contribuyó en menor medida, dado que la maniobra generó una activación simpática marcada pero sin un aumento sostenido de la frecuencia cardiaca.

**Análisis 2: Alcance y limitaciones del sistema para cuantificar el dolor percibido**

El sistema calcula el SPI a partir de dos variables periféricas, la amplitud pico-valle (PPGA) y el intervalo entre latidos (HBI), que se modulan con la activación simpática asociada a un estímulo nociceptivo. Esto significa que el índice refleja el nivel de activación autonómica y no el dolor en sí mismo, debido a que otras fuentes de activación simpática (frío, esfuerzo de sujeción del hielo, movimiento, incluso la anticipación del estímulo) pueden producir el mismo tipo de respuesta. El valor máximo de SPI registrado justo en la transición entre la maniobra y la recuperación coincide con el momento en que el sujeto suelta el hielo, un instante asociado a un cambio brusco de postura y tensión muscular en todo el cuerpo (incluida la mano que sostiene el sensor), por lo que el pico probablemente refleja también un artefacto de movimiento y no únicamente un cambio fisiológico.

Dentro de este alcance, el sistema cumplió su función principal: capturar de forma continua la señal PPG y traducirla en un índice que sigue el curso esperado de la respuesta simpática ante un estímulo doloroso controlado, sin necesidad de instrumentación clínica adicional. Para fortalecer su uso como herramienta de cuantificación, el siguiente paso natural sería contrastarlo con un monitor de signos vitales certificado y ampliar la evaluación a más de un sujeto de prueba.

### 4. Preguntas para la Discusión

**Pregunta 1: ¿Cómo se relacionan las variaciones del volumen sanguíneo periférico con el balance autonómico?**

El volumen de sangre que llega a los tejidos periféricos en cada latido está regulado por el tono vasomotor de las arteriolas, controlado en gran medida por fibras simpáticas vasoconstrictoras. Ante un aumento de la actividad simpática, como el inducido por el CPT, se produce **vasoconstricción periférica**, lo que reduce el volumen de sangre que ingresa al lecho vascular con cada latido y disminuye la amplitud de la onda de pulso (PPGA); de forma simultánea, la retirada del tono vagal cardiaco acorta el intervalo entre latidos (HBI), reflejando el aumento de frecuencia cardiaca asociado a la activación simpática. Un predominio parasimpático, en cambio, se traduce en vasodilatación, mayor amplitud de pulso e intervalos entre latidos más largos.

Esta relación entre amplitud y frecuencia del pulso es precisamente la que el SPI utiliza al combinar PPGA y HBI en un único índice: al normalizar y ponderar ambas variables, el sistema traduce cambios vasculares y cronotrópicos, que de otro modo habría que interpretar por separado, en una sola medida continua del balance autonómico.

**Pregunta 2: ¿Cómo se compara el SPI con otros índices comúnmente empleados en cirugía, como el índice nocicepción-analgesia (ANI) y el índice de perfusión?**

El SPI, el ANI y el índice de perfusión (PI) parten de señales periféricas ya disponibles en quirófano, pero cada uno enfatiza una variable fisiológica distinta. El SPI combina amplitud de pulso e intervalo entre latidos derivados de la PPG, capturando el componente vasomotor y el cronotrópico de la respuesta simpática. El **ANI** se calcula a partir de la variabilidad de la frecuencia cardiaca asociada a la respiración, reflejando principalmente el **tono vagal cardiaco**: valores altos indican predominio parasimpático (buena analgesia) y valores bajos indican retirada vagal por nocicepción, es decir, evalúa el balance desde la rama parasimpática, complementaria a la que enfatiza el SPI. El **índice de perfusión (PI)** se obtiene de la relación entre el componente pulsátil y no pulsátil de la señal del oxímetro, siendo un indicador puramente vasomotor, sin componente de frecuencia cardiaca, por lo que resulta sensible también a factores como la temperatura periférica o el uso de vasoactivos.

Los tres índices resultan complementarios entre sí más que redundantes: el SPI y el PI comparten la dependencia del tono vasomotor periférico (de hecho, el PI puede entenderse como un componente parcial de lo que el SPI ya incorpora), mientras que el ANI aporta información adicional desde la rama vagal. Por esta razón, su uso conjunto en un mismo monitor puede ofrecer una lectura más completa del balance nocicepción-analgesia que cualquiera de los tres de forma aislada.

> [!NOTE]
> A diferencia del SPI y el PI, que dependen del tono vasomotor simpático, el ANI se basa en la **arritmia sinusal respiratoria** (variabilidad de HBI ligada a la respiración), por lo que responde principalmente a cambios en el tono parasimpático cardiaco.

### 5. Conclusiones

El uso de la onda de pulso como fuente de información sobre el balance entre la nocicepción y la analgesia responde a la necesidad de contar, durante la anestesia general, con una medida objetiva y continua del estado autonómico del paciente, que no dependa de su reporte verbal ni de instrumentación adicional a la ya disponible en el quirófano. En esta práctica se implementó un sistema ambulatorio, basado en el sensor MAX30102 acoplado a un ESP32, capaz de adquirir la señal fotopletismográfica, extraer de ella la amplitud pico-valle y el intervalo entre latidos mediante el método del alpinista, y calcular con ellos el índice pletismográfico quirúrgico (SPI) en tiempo real. Al someter al sistema a la prueba de Cold Pressor Test como estímulo doloroso controlado, sosteniendo el sujeto de prueba un trozo de hielo con la mano contraria a la que tenía apoyada sobre el sensor, el SPI siguió el comportamiento esperado a lo largo de las tres fases del protocolo: valores dentro del rango de referencia durante el reposo inicial, un ascenso sostenido hacia el final de la línea base y durante la maniobra, y un pico marcado coincidente con el cierre del estímulo frío, seguido de un descenso progresivo durante la recuperación. Este resultado permite concluir que el sistema desarrollado logra aproximar, con electrónica de bajo costo y sin necesidad de un monitor comercial certificado, el mismo principio de cálculo que emplean los equipos especializados de anestesia para estimar el balance nocicepción-analgesia, cumpliendo así con el objetivo general planteado para la práctica. Como siguiente paso, sería conveniente contrastar las lecturas del sistema frente a un monitor de signos vitales certificado y ampliar la prueba a varios sujetos, de manera que se pueda precisar con mayor detalle la contribución específica del estímulo nociceptivo frente a otras fuentes de activación simpática (esfuerzo muscular, movimiento del sensor) presentes durante el registro.

### Referencias Bibliográficas

[1] V. Bonhomme, K. Uutela, G. Hans, I. Maquoi, J. D. Born y J. F. Brichant, "Comparison of the Surgical Pleth Index™ with haemodynamic variables to assess nociception-anti-nociception balance during general anaesthesia," British Journal of Anaesthesia, vol. 106, no. 1, pp. 101–111, 2011. https://doi.org/10.1093/bja/aeq291.

[2] M. Huiku et al., "Assessment of surgical stress during general anaesthesia," British Journal of Anaesthesia, vol. 98, no. 4, pp. 447–455, 2007. https://doi.org/10.1093/bja/aem004.

[3] "Surgical pleth index monitoring in perioperative pain management: usefulness and limitations," Korean Journal of Anesthesiology, ekja.org/upload/pdf/kja-23158.pdf.

[4] GE Healthcare, "Surgical Pleth Index — Quick Guide" y documentación técnica asociada, gehealthcare.co.uk.
