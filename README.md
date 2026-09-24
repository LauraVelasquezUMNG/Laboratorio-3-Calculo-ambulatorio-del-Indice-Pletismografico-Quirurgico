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

La fotopletismografía (PPG) permite registrar de forma no invasiva las variaciones del volumen sanguíneo periférico [1]. De la onda de pulso se extraen características como el intervalo entre pulsos y la amplitud, con las que se calcula el índice pletismográfico quirúrgico (SPI), un índice que estima el balance entre la nocicepción y la analgesia durante la anestesia general [2].

El objetivo de la Parte A fue construir el sistema de adquisición de la onda de pulso y definir el diseño experimental para inducir una respuesta fisiológica similar a la del dolor agudo, mediante la técnica Cold Pressor Test (CPT).

### **Circuito de acondicionamiento y sensor**

Se montó en protoboard el circuito de la Figura 1 de la guía el cual consta de:

- Etapa de emisión: un transistor 2N3904 que controla el LED emisor del optoacoplador.
- Etapa de detección: el fototransistor, cuya señal pasa por un filtro pasa-altas pasivo que elimina el nivel DC.
- Etapa de amplificación y filtrado: un filtro pasa-bajas activo con LM358 y un segundo amplificador, con potenciómetros para ajustar el offset y la amplitud.

Como sensor se usó el optoacoplador, modificado para funcionar como sensor de reflectancia. El emisor y el detector se separaron y se colocaron lado a lado, de modo que la luz reflejada por el dedo llegue al detector. La salida del circuito se conectó a una entrada analógica de una ESP32, y se verificó la captura de las variaciones del volumen sanguíneo. Aunque el circuito se montó y se probó, la captura final se hizo con un módulo MAX30102 conectado a la ESP32. Este módulo integra en un solo encapsulado LEDs rojo e infrarrojo, un fotodetector y un ADC interno.

### **Cold Pressor Test (CPT)**

El CPT es una prueba de estrés que consiste en exponer una parte del cuerpo, clásicamente la mano o el antebrazo sumergidos en agua helada, a una temperatura muy baja durante un tiempo definido. El frío activa los nociceptores y termorreceptores cutáneos y provoca una respuesta simpática. Esta respuesta produce vasoconstricción periférica, aumento de la frecuencia cardíaca y de la presión arterial, y una sensación de dolor que aumenta con el tiempo [3]. Por eso el CPT se usa en investigación como estímulo doloroso controlado, reproducible y seguro. En el contexto del SPI, ante un estímulo nociceptivo, la activación simpática reduce la amplitud del pulso por la vasoconstricción y acorta el intervalo entre latidos. Ambos cambios elevan el SPI. Por eso se espera un aumento durante el CPT y un regreso hacia el valor basal en la recuperación. 

Para aplicarlo en el laboratorio, en lugar de la inmersión en agua helada, se usó una variante simple: se le pidió al profesor, que fue el sujeto de prueba, que sostuviera un trozo de hielo en la mano izquierda, mano contraria del sensor. 
