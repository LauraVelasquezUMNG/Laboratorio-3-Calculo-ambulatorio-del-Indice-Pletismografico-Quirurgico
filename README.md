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

### **Cold Pressor Test (CPT)**

El CPT es una prueba de estrés que consiste en sumergir una parte del cuerpo, clásicamente la mano o el antebrazo, en agua helada durante un tiempo definido. El frío activa los nociceptores y termorreceptores cutáneos y provoca una respuesta simpática, la cual se traduce en vasoconstricción periférica, aumento de la frecuencia cardíaca y de la presión arterial, y una sensación de dolor que se incrementa con el tiempo [3]. Por esta razón, el CPT se emplea en investigación como estímulo doloroso controlado, reproducible y seguro. En el contexto del SPI, la activación simpática inducida por un estímulo nociceptivo reduce la amplitud del pulso por vasoconstricción y acorta el intervalo entre latidos; ambos cambios elevan el valor del SPI. Por eso se espera un aumento durante el CPT y un retorno hacia el valor basal durante la recuperación.

Para aplicarlo en el laboratorio, en lugar de la inmersión en agua helada se empleó una variante simple: se le pidió al profesor, quien actuó como sujeto de prueba, que sostuviera un trozo de hielo en la mano izquierda, opuesta a la que portaba el sensor. La captura tuvo una duración total de 2 minutos, distribuida en tres fases: fase basal (0-40 s), en reposo y sin estímulo; fase de CPT (40-80 s), durante la cual el sujeto sostiene el hielo; y fase de recuperación (80-120 s), en la que se retira el hielo y el sujeto vuelve al reposo.

> ### Parte B

### **Revisión literaria: Definición matemática del índice pletismográfico quirúrgico (SPI)**

#### **1. Fundamento fisiológico**

El SPI fue desarrollado por GE Healthcare y descrito por primera vez por Huiku et al. en 2007, bajo el nombre inicial de Surgical Stress Index (SSI) [1]. Su propósito es construir una medida continua y objetiva del balance entre la estimulación nociceptiva (dolor) y el efecto analgésico durante la anestesia general, a partir de una señal ya disponible en el quirófano: la onda de pulso obtenida por pulsioximetría [2]. Esto evita instrumentación adicional, pues reutiliza el sensor de SpO₂ que casi todo paciente anestesiado ya lleva puesto.

Un estímulo doloroso activa el sistema nervioso simpático, lo que produce dos efectos medibles en la periferia:

- Vasoconstricción periférica, que reduce el volumen de sangre que entra al tejido con cada latido y, en consecuencia, disminuye la amplitud de la onda PPG (PPGA).
- Aumento de la frecuencia cardíaca, que acorta el intervalo entre latidos (HBI).

La analgesia (por ejemplo, con opioides) tiende a atenuar ambos efectos. Por esta razón, el SPI combina PPGA y HBI: son dos indicadores fisiológicos distintos del mismo fenómeno, uno más ligado al tono vasomotor simpático (PPGA) y el otro más ligado al efecto de fármacos opioides sobre el nodo sinusal (HBI).

#### **2. Variables de entrada**

- PPGA (Photoplethysmographic Pulse Wave Amplitude): diferencia entre el valor máximo y el mínimo de la señal PPG en cada latido (la componente AC de la onda de pulso).
- HBI (Heart Beat Interval): intervalo de tiempo entre dos picos sistólicos consecutivos; equivale al recíproco de la frecuencia cardíaca instantánea.

#### **3. Normalización**

Dado que la amplitud absoluta de la PPG y la frecuencia cardíaca basal varían considerablemente entre personas, ninguna de las dos variables se emplea en su forma bruta. GE Healthcare aplica una transformación de histograma sobre una ventana móvil de valores recientes de PPGA y HBI: cada nueva muestra se reubica según el percentil que ocupa dentro de la distribución de valores anteriores del mismo paciente, generando así PPGAnorm y HBInorm, ambos acotados entre 0 y 100 [4]. Esto es lo que permite comparar el SPI entre pacientes distintos, en lugar de limitarse a comparar cambios relativos dentro de un mismo paciente.

Al iniciar la monitorización, el algoritmo requiere un período de "aprendizaje" (cercano a 3 minutos) para construir esa distribución de valores basales antes de que el número de SPI sea confiable; antes de ese punto, el valor se muestra en gris.

#### **4. Fórmula matemática e interpretación del SPI**

$$SPI = 100 - (0.7 \times PPGA_{norm} + 0.3 \times HBI_{norm})$$

Esta es la fórmula original reportada por Huiku et al. (2007) y confirmada en la documentación técnica de GE Healthcare [4]. El peso de 0,7 sobre PPGAnorm frente a 0,3 sobre HBInorm refleja que la amplitud del pulso responde de forma más marcada y más rápida al estímulo nociceptivo que el intervalo entre latidos.

El SPI es un número adimensional entre 0 y 100. Valores altos reflejan mayor actividad simpática o nocicepción (menor PPGAnorm y/o menor HBInorm), mientras que valores bajos reflejan analgesia adecuada o ausencia de estímulo doloroso. El rango de referencia para una anestesia bien balanceada en adultos sanos es 20-50, y se recomienda evitar incrementos súbitos mayores a 10 puntos, ya que estos son más indicativos de un evento nociceptivo agudo que el valor absoluto en sí [4].

Cabe señalar que el SPI, tal como lo definió GE Healthcare, está pensado para un paciente bajo anestesia general monitoreado con un pulsioxímetro clínico certificado, y emplea una ventana de normalización basada en varios minutos de datos del mismo sujeto. En este laboratorio se aplicó el mismo principio de cálculo (amplitud de pulso e intervalo entre latidos) a una señal PPG adquirida con un MAX30102 sobre la ESP32, en una persona consciente y en reposo, sin el algoritmo propietario de normalización histográfica del monitor comercial.

### Referencias Bibliográficas

[1] V. Bonhomme, K. Uutela, G. Hans, I. Maquoi, J. D. Born y J. F. Brichant, "Comparison of the Surgical Pleth Index™ with haemodynamic variables to assess nociception-anti-nociception balance during general anaesthesia," British Journal of Anaesthesia, vol. 106, no. 1, pp. 101–111, 2011. https://doi.org/10.1093/bja/aeq291.

[2] M. Huiku et al., "Assessment of surgical stress during general anaesthesia," British Journal of Anaesthesia, vol. 98, no. 4, pp. 447–455, 2007. https://doi.org/10.1093/bja/aem004.

[3] "Surgical pleth index monitoring in perioperative pain management: usefulness and limitations," Korean Journal of Anesthesiology, ekja.org/upload/pdf/kja-23158.pdf.

[4] GE Healthcare, "Surgical Pleth Index — Quick Guide" y documentación técnica asociada, gehealthcare.co.uk.
