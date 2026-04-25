# Sistemas-embebidos-2
 Generador de Gráficos Vectoriales XY con Arduino y Filtros RC
 
 Este proyecto demuestra la generación de gráficos vectoriales en un osciloscopio utilizando señales PWM (Modulación por Ancho de Pulsos) procesadas mediante filtros analógicos. El sistema permite "dibujar" tres árboles de Navidad en una pantalla en modo XY, controlando la velocidad de refresco mediante un potenciómetro.

Método Utilizado: Síntesis Analógica de Señales

El proyecto se basa en la conversión de señales digitales rápidas en voltajes analógicos variables:

- Mapeo de Coordenadas: Se definen arreglos de bytes que representan los puntos (vértices) de la figura en un plano cartesiano.
- Generación de Voltaje (PWM): El Arduino Nano genera señales PWM en los pines D5 (Y) y D6 (X).
- Filtro Paso Bajo (RC): Las señales PWM de alta frecuencia pasan por una red de resistencia (1kΩ) y capacitor (0.5µF). Esto elimina los picos digitales y entrega un voltaje promedio suave.
- Visualización XY: Al conectar el Canal X al eje horizontal y el Canal Y al vertical, el haz de luz del osciloscopio se mueve según los voltajes, recreando la imagen.

Materiales y Requisitos

Hardware (Simulado en Proteus):


- 1x Arduino Nano (Atmega328P).
- 2x Resistencias de 1kΩ (Filtro).
- 2x Capacitores de 0.5µF (Filtro).
- 1x Potenciómetro de 10kΩ (Control de velocidad).

Software:


- Arduino IDE (Compilación).
- Proteus 8 Professional (Simulación).

Problemas Presentados y Soluciones

  - Problema
- Imagen con "ruido" o rayas internas
- Figura deformada o circular
- Dibujo cortado o fuera de rango
- Líneas de "salto" muy visibles

  - Causa Tecnica
- Capacitancia insuficiente (100nF) que dejaba pasar el rizado del PWM
- El tiempo de carga/descarga del capacitor (RC) era mayor al tiempo de cambio de punto.
- Las coordenadas sumadas excedían los 255 niveles del PWM (5V).
- El osciloscopio dibuja el trayecto mientras el haz viaja de una figura a otra.

  - Solución Aplicada
- Se incrementó el valor a 0.47µF, logrando un trazo sólido y limpio.
- Se ajustó el point_delay mediante el potenciómetro para equilibrar velocidad y definición.
- Se recalcularon los offsets de posicionamiento para mantener los 3 árboles dentro del rango visible.
- Se optimizó el orden de dibujo en el código para minimizar la distancia de los saltos.

 Código Fuente (Arduino):

     //Inputs/Outputs
    int X_pin = 6;     
    int Y_pin = 5;     
    int Pot = A0;
    
    int point_delay = 1000;         
    #define how_many_vertices  19   
    
    // Coordenadas originales
    byte x_axis[how_many_vertices] = {9,9,3,9,4,9,6,9,8,10,12,11,14,11,16,11,17,11,11};
    byte y_axis[how_many_vertices] = {3,6,6,10,10,14,14,17,17,19,17,17,14,14,10,10,6,6,3};
    
    void setup() {
      pinMode(X_pin,OUTPUT);      
      pinMode(Y_pin,OUTPUT);      
      pinMode(Pot,INPUT);         
    
      TCCR0A = (TCCR0A & B10100011) | B10100011;
      TCCR0B = (TCCR0B & B00000001) | B00000001;
      TIMSK0 = (TIMSK0 & B11111000);
    }
    
    void loop() {
      point_delay = map(analogRead(Pot), 0, 1024, 10, 1000); // Delay más corto para compensar el 3er árbol
    
      // --- ÁRBOL 1 (Izquierda - Grande) ---
      for(int i = 0; i < how_many_vertices; i++) {
        analogWrite(X_pin, x_axis[i]); 
        analogWrite(Y_pin, y_axis[i]);
        delayMicroseconds(point_delay);
      }
    
      // --- ÁRBOL 2 (Centro - PEQUEÑO) ---
      for(int i = 0; i < how_many_vertices; i++) {
        // Escala al 60% y desplazamiento para centrarlo
        analogWrite(X_pin, (x_axis[i] * 0.6) + 18); 
        analogWrite(Y_pin, (y_axis[i] * 0.6));
        delayMicroseconds(point_delay);
      }
    
      // --- ÁRBOL 3 (Derecha - Grande) ---
      for(int i = 0; i < how_many_vertices; i++) {
        // Desplazado a la derecha
        analogWrite(X_pin, x_axis[i] + 30); 
        analogWrite(Y_pin, y_axis[i]);
        delayMicroseconds(point_delay);
      }
    }












