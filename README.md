# Regadera Automatizada con Energía Solar e Integración Web 3.0

## 1. Introducción y Objetivos
Este proyecto consiste en un sistema de riego automatizado para optimizar el uso de recursos hídricos en agricultura urbana o jardines comunitarios. El sistema utiliza energía solar fotovoltaica para alimentar una bomba de agua controlada por un microcontrolador ESP32.

### Objetivos:
* Monitorear el nivel del contenedor de agua para evitar que la bomba trabaje en seco.
* Automatizar el riego utilizando energía 100% limpia y renovable.
* Registrar de forma inmutable los datos de ahorro de agua en la Blockchain para certificar prácticas ecológicas.

---

## 2. Componentes de Hardware y Conexiones
Para la implementación física del sistema se seleccionaron los siguientes dispositivos electrónicos:

* **Controlador principal:** ESP32 (con conectividad Wi-Fi integrada).
* **Sensor de nivel de agua:** Sensor ultrasónico HC-SR04 (mide la distancia al espejo de agua en el tanque de reserva).
* **Actuador:** Bomba de agua sumergible de 5V.
* **Alimentación:** Panel solar con batería recargable y módulo de carga.
* **Alarma visual/sonora:** Buzzer activo de 5V para alertas locales.

### Tabla de Conexiones de Pines:
| Componente | Pin del Dispositivo | Pin en el ESP32 | Descripción |
| :--- | :--- | :--- | :--- |
| Sensor HC-SR04 | Trigger | GPIO 23 | Pulso de disparo de ultrasonido |
| Sensor HC-SR04 | Echo | GPIO 22 | Recepción de eco de retorno |
| Relé / Bomba | Señal de Control | GPIO 18 | Encendido/Apagado de la bomba |
| Buzzer Alarma | VCC | GPIO 5 | Alerta de tanque vacío |

---

## 3. Firmware de Programación (C++)
Código fuente en C++ desarrollado para compilar en el entorno de Arduino IDE para el control local del riego:

```cpp
const int trigPin = 23;
const int echoPin = 22;
const int bombaPin = 18;
const int buzzerPin = 5;

const float velocidadSonido = 0.0343; 
const float alturaTanque = 100.0; // Altura total del tanque en cm

void setup() {
  Serial.begin(115200);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(bombaPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);
}

void loop() {
  // Medición de distancia por ultrasonido
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duracion = pulseIn(echoPin, HIGH);
  float distancia = (duracion * velocidadSonido) / 2;
  float nivelAgua = alturaTanque - distancia;

  Serial.print("Nivel de agua en el tanque: ");
  Serial.print(nivelAgua);
  Serial.println(" cm");

  // Si el tanque tiene agua suficiente (más de 15 cm), permite el riego.
  // Si está casi vacío (menos de 15 cm), apaga la bomba por seguridad y activa alarma.
  if (nivelAgua < 15.0) {
    digitalWrite(bombaPin, LOW);   // Apaga la bomba (protección contra trabajo en seco)
    digitalWrite(buzzerPin, HIGH);  // Alarma activa
  } else {
    digitalWrite(bombaPin, HIGH);  // Activa el riego
    digitalWrite(buzzerPin, LOW);   // Alarma desactivada
  }

  delay(5000); // Muestreo cada 5 segundos
}
