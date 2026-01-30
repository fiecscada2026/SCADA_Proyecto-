# Simulación y estudio de estabilidad en un sistema de potencia de 8 barras

Simulación y estudio de estabilidad en un sistema de potencia de 8 barras con dos unidades síncronas usando plataforma SCADA aplicada a sistemas de potencia. 

---

## 2. Objetivo del ejercicio

El objetivo del ejercicio es analizar el comportamiento dinámico y la estabilidad de un sistema de potencia de 8 barras ante variaciones de carga y eventos de falla, utilizando un modelo detallado con dos máquinas síncronas, sus sistemas de excitación y turbinas hidráulicas, e implementando un esquema tipo SCADA con niveles de proceso, RTU/IED y Master/HMI. 

---

## 3. Descripción del sistema implementado

El sistema de potencia modelado se basa en un sistema estándar de 8 barras conectado a una red de 230 kV, en el cual operan dos generadores síncronos de 200 MVA y 13.8 kV, cada uno con su sistema de excitación, turbina hidráulica y gobernador.  
El modelo permite estudiar los transitorios de voltaje, corriente, velocidad de rotor y ángulo del rotor cuando se aplican fallas en diferentes puntos de la red o cuando se conectan y desconectan cargas mediante interruptores automáticos. 
Las cargas principales (LOAD1, LOAD2 y LOAD3) se alimentan a través de un transformador elevador que lleva el voltaje de 13.8 kV a 230 kV, y se conectan a distintas barras mediante líneas de transmisión y breakers controlados por señales lógicas SW1, SW2 y SW3. 
Adicionalmente, se pueden provocar fallas entre fases y fallas monofásicas a tierra mediante las señales SW4 y SW5, lo que genera perturbaciones significativas sobre las magnitudes eléctricas y las variables mecánicas de los generadores. 

---

## 4. Diagrama de conexiones (hardware y software)

A nivel de modelado en Simulink, el sistema se organiza en tres grandes subsistemas: Nivel de Proceso, RTU/IED y Master/HMI, interconectados por señales de medición y control.  

- **Nivel de Proceso:** contiene el sistema de potencia de 8 barras, los generadores síncronos, los transformadores, las líneas y las cargas, así como los puntos de medición de voltajes y corrientes trifásicas (VabcB1–VabcB8, IabcB1–IabcB8) y las variables mecánicas wm y θ de cada unidad.
- **Subsistema RTU/IED:** recibe las mediciones del nivel de proceso, calcula magnitudes RMS y potencias activa y reactiva, y transmite esta información hacia el nivel maestro, además de recibir desde allí las órdenes de maniobra SW1–SW5.  
- **Master/HMI:** concentra la información de los IED, muestra tendencias de voltajes a 13.8 kV y 230 kV, velocidades y ángulos de rotor, y potencias activa y reactiva; además, permite operar los breakers de cargas y activar las fallas desde un panel frontal. 

En el contexto de simulación en tiempo real, el modelo completo se integra a RT‑Lab/OPAL‑RT, donde se compila y ejecuta el código, mientras que la interfaz HMI actúa como “front‑end” tipo SCADA para supervisión y control. 

---

## 5. Explicación detallada de las conexiones

En el **Nivel de Proceso**, las señales de control SW1, SW2 y SW3 gobiernan la apertura o cierre de los breakers asociados a LOAD1, LOAD2 y LOAD3, permitiendo representar incrementos o pérdidas de carga en distintos instantes de tiempo. 
Las señales SW4 y SW5 controlan bloques de falla que introducen contingencias en líneas específicas, como cortocircuitos entre dos fases o fallas monofásicas a tierra; estas fallas provocan caídas bruscas de voltaje y oscilaciones marcadas en velocidad y ángulo del rotor. 

El **Subsistema RTU/IED** toma como entradas las mediciones trifásicas de tensión y corriente en las barras (VabcB#, IabcB#) y las procesa para obtener valores RMS, potencias PB# y QB# y otros indicadores eléctricos que luego se envían al maestro.   
En sentido inverso, las órdenes de operación generadas desde la HMI (cambios de estado en LOAD1–LOAD3 o activación de fallas L4‑6 y L1‑3) se transmiten como señales digitales hacia los IED y de allí al nivel de proceso, cerrando el lazo de supervisión y control.  

En el **Master/HMI**, los datos provenientes de los IED se conectan a gráficos de tendencia para voltajes en buses de 230 kV (B1, B2, B3, B4, B6, B7) y de 13.8 kV (B5, B8), a indicadores de frecuencia y ángulo de los generadores, y a visualizaciones de potencias activa y reactiva en los distintos buses. 
El panel frontal incluye además un panel de control con botones o conmutadores para LOAD1, LOAD2, LOAD3 y para las fallas L4‑6 y L1‑3, donde cada elemento indica su estado (por ejemplo, colores verde/rojo para cargas conectadas o desconectadas). 

---

## 6. Procedimiento de simulación

1. **Configuración del modelo base**  
   Se construye el sistema de 8 barras en Simulink con dos generadores síncronos de 200 MVA/13.8 kV, sus sistemas de excitación, turbinas hidráulicas y gobernadores, el transformador elevador a 230 kV, las líneas de transmisión y las cargas LOAD1–LOAD3, incorporando mediciones de VabcB# e IabcB#. 

2. **Implementación de niveles SCADA**  
   Se agrupan los elementos en los subsistemas de Nivel de Proceso, RTU/IED y Master/HMI; en el RTU/IED se añaden bloques para cálculo RMS y potencias, y se definen las señales de mando SW1–SW5 que gobiernan breakers y fallas. 

3. **Simulación en entorno offline**  
   Se ejecutan primeras simulaciones sin fallas ni maniobras de carga para observar las tendencias de voltajes, velocidades y ángulos del rotor de G1 y G2 en régimen estable. 

4. **Simulaciones con variación de carga y fallas**  
   Se programan secuencias donde se abren o cierran progresivamente los breakers de LOAD1, LOAD2 y LOAD3, y posteriormente se introducen fallas de dos fases y fase‑tierra mediante SW4 y SW5, registrando el efecto sobre las variables eléctricas y mecánicas. 

5. **Carga del modelo en RT‑Lab/OPAL‑RT**  
   El modelo se importa a un nuevo proyecto en RT‑Lab, se agregan bloques específicos como `OpComm` y bloques de memoria para la comunicación con el simulador en tiempo real, y se realiza el proceso de compilación (`build`) hasta obtener el estado apto para carga. 

6. **Ejecución en tiempo real**  
   Una vez compilado, el modelo se carga al simulador con la opción “Load” y se controla con las acciones “Execute”, “Pause” y “Reset”, mientras la HMI muestra las señales en tiempo real y permite aplicar las mismas secuencias de cargas y fallas que en las simulaciones offline.

---
Tutorial:
https://drive.google.com/file/d/1YYBb3h0N_pDngssKdtj50yb2vTkC173J/view?usp=drive_link

---

## 7. Resultados obtenidos

En la primera simulación sin fallas ni maniobras, las gráficas de tendencia muestran voltajes estables en las barras de 230 kV y 13.8 kV, así como velocidades y ángulos de rotor cercanos a la referencia de operación, evidenciando un régimen estacionario estable.
En la segunda simulación, durante 40 segundos se analizan las respuestas ante desconexión y reconexión de cargas, observándose pequeñas perturbaciones transitorias en los voltajes y variaciones en los ángulos de rotor y velocidades de las máquinas, que luego convergen nuevamente a la referencia de 1 p.u. gracias a la acción de los gobernadores.
Cuando se aplican fallas (entre fases y fase‑tierra), las formas de onda muestran caídas abruptas de voltaje en las barras afectadas y oscilaciones de mayor amplitud en velocidad y ángulo de los generadores, con picos elevados en las corrientes del estator de G1 y G2.  

Las tablas de resultados incluyen valores RMS de voltajes y corrientes en cada barra (VabcB#rms, IabcB#rms) y las potencias activa y reactiva PB#/QB#, donde se aprecia la distribución de flujos de potencia en la red bajo las diferentes condiciones de operación. 
En la ejecución en tiempo real, las capturas de pantalla muestran que las tendencias y oscilaciones observadas en RT‑Lab son consistentes con las simulaciones offline, confirmando que el modelo se ha portado correctamente al simulador de tiempo real. 

---

## 8. Análisis de resultados

Los resultados de la primera simulación demuestran que, en ausencia de perturbaciones, el sistema de 8 barras mantiene un comportamiento estable, con voltajes y velocidades de rotor prácticamente constantes y ángulos de rotor que evolucionan suavemente dentro de márgenes aceptables.
Las maniobras de carga generan oscilaciones moderadas en las variables eléctricas y mecánicas, pero los controladores de las turbinas y los sistemas de excitación actúan de manera efectiva, amortiguando los transitorios y permitiendo que el sistema retorne a un nuevo punto de equilibrio.
Las simulaciones de falla evidencian un impacto mucho más severo sobre la estabilidad: los voltajes caen bruscamente, las corrientes aumentan de forma significativa y los ángulos de rotor presentan oscilaciones pronunciadas, lo que permite ilustrar condiciones cercanas a la pérdida de sincronismo si las fallas se mantienen por tiempos prolongados. 
El uso del subsistema IED para calcular potencias y magnitudes RMS, junto con la HMI para visualizar tendencias, facilita la interpretación de estos fenómenos y acerca el modelo a la realidad de un sistema SCADA industrial. 

---

## 9. Conclusiones del ejercicio

El ejercicio permitió implementar un modelo detallado de un sistema de potencia de 8 barras con dos unidades de generación síncrona, adecuado para estudiar fenómenos de estabilidad transitoria ante cambios de carga y fallas en la red. [file:48]  
La estructura multinivel (Proceso – RTU/IED – Master/HMI) demostró la funcionalidad de una arquitectura SCADA aplicada a sistemas de potencia, donde las mediciones de campo son procesadas por IED y presentadas en una interfaz gráfica que permite supervisión y control en tiempo real. [file:48]  
Las simulaciones offline y en tiempo real mostraron comportamientos coherentes, validando la correcta implementación del modelo en RT‑Lab/OPAL‑RT y evidenciando la utilidad de estas herramientas para el análisis de estabilidad y la formación en operación de sistemas eléctricos. [file:48]  
En conjunto, el trabajo refuerza conceptos de estabilidad, control de generación, respuesta ante contingencias y uso de plataformas de simulación en tiempo real integradas con entornos tipo SCADA.
