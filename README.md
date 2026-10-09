# 🔌 Reconocimiento de Tramas I²C con Analizador Lógico

## 📌 Descripción

Este laboratorio tiene como propósito identificar y analizar la estructura de las tramas del protocolo de comunicación I²C mediante una **Raspberry Pi Pico 2W**, una pantalla OLED SSD1306 y un analizador lógico utilizando el software Logic 2.

Se realizan capturas y decodificaciones de las señales digitales para reconocer las condiciones START y STOP, el direccionamiento de 7 bits, los estados de reconocimiento ACK/NACK y los comandos transmitidos a la pantalla OLED mediante MicroPython.

## 🎯 Objetivos

* Identificar las condiciones START y STOP, el direccionamiento de 7 bits y los estados ACK/NACK en las tramas I²C mediante Logic 2.
* Verificar la respuesta de la pantalla OLED SSD1306 utilizando direcciones correctas e incorrectas y comparar las capturas obtenidas.
* Determinar la frecuencia de reloj SCL mediante la medición del período de la señal adquirida con el analizador lógico.
* Implementar el escaneo de direcciones I²C mediante MicroPython para detectar los dispositivos conectados al bus.
* Reconocer los comandos y datos transmitidos a la pantalla OLED, relacionando los códigos hexadecimales observados con sus respectivas funciones.

## 🛠️ Materiales y herramientas

* **Raspberry Pi Pico 2W:** microcontrolador encargado de generar las transacciones I²C.
* **Pantalla OLED SSD1306:** dispositivo periférico con dirección I²C `0x3C`.
* **Analizador lógico USB de 8 canales:** utilizado para capturar las señales digitales del bus.
* **Logic 2:** software empleado para visualizar y decodificar las tramas.
* **MicroPython:** lenguaje utilizado para implementar las pruebas de comunicación.
* Cables de conexión y referencia común de tierra (GND).

## 🔗 Configuración del bus I²C

| Parámetro                          | Configuración                   |
| ---------------------------------- | ------------------------------- |
| Interfaz                           | I²C1                            |
| Pin SCL                            | GP15                            |
| Pin SDA                            | GP14                            |
| Dirección OLED                     | `0x3C`                          |
| Frecuencia configurada inicial     | 100 kHz                         |
| Frecuencia del menú OLED           | 50 kHz                          |
| Canal CH0 del analizador           | SCL                             |
| Canal CH1 del analizador           | SDA                             |
| Frecuencia de muestreo recomendada | ≥ 1 MS/s para un bus de 100 kHz |

La tierra del analizador lógico debe conectarse a GND de la Raspberry Pi Pico 2W para obtener una referencia común de las señales.

## 📡 Fundamentos de una trama I²C

Una transacción de escritura básica se representa mediante la siguiente secuencia:

`START → Dirección + R/W → ACK/NACK → STOP`

* **START:** SDA pasa de nivel alto a bajo mientras SCL permanece alto.
* **Dirección:** identifica el dispositivo con el que se establece la comunicación.
* **R/W:** indica si la operación corresponde a lectura o escritura.
* **ACK:** el dispositivo receptor reconoce la comunicación llevando SDA a nivel bajo durante el noveno pulso de reloj.
* **NACK:** SDA permanece en nivel alto durante el noveno pulso cuando no se reconoce la dirección.
* **STOP:** SDA pasa de nivel bajo a alto mientras SCL permanece alto.

Para la dirección `0x3C`, una operación de escritura transmite el octeto `0x78`. Al utilizar la dirección `0x3D`, el octeto transmitido es `0x7A`.

## 🧪 Procedimiento experimental

### 1. Montaje y configuración

Se conectó la Raspberry Pi Pico 2W con la pantalla OLED SSD1306 y el analizador lógico. Se configuró la interfaz I²C1 utilizando GP15 como SCL y GP14 como SDA. En Logic 2 se habilitó el decodificador I²C para observar las transacciones.

### 2. Pruebas de reconocimiento ACK/NACK

Se ejecutó el programa `OLED_ADDR_test.py` para comprobar la respuesta de la pantalla con la dirección correcta `0x3C`. Posteriormente, se modificó la dirección a `0x3D` para observar el comportamiento de una dirección no reconocida.

Se compararon las capturas obtenidas, identificando el octeto transmitido y el estado de SDA durante el noveno pulso de reloj.

### 3. Escaneo de direcciones I²C

Mediante el programa `scan_i2c_addr.py` se utilizó el método `i2c.scan()` de MicroPython para detectar los dispositivos conectados al bus. Los resultados de la consola se contrastaron con las capturas del analizador lógico.

### 4. Análisis de comandos de la pantalla OLED

Se ejecutó el programa `OLED_demo_menu.py` para transmitir diferentes comandos al controlador SSD1306. Se analizaron las transacciones correspondientes al apagado, encendido, ajuste del contraste, inversión de imagen y visualización de texto o animaciones.

## 📊 Resultados experimentales

### Pruebas ACK/NACK

| Parámetro                | Dirección `0x3C` | Dirección `0x3D` |
| ------------------------ | ---------------- | ---------------- |
| Octeto transmitido       | `0x78`           | `0x7A`           |
| Reconocimiento           | ACK              | NACK             |
| Nivel de SDA en el bit 9 | Bajo             | Alto             |
| Condiciones START y STOP | Detectadas       | Detectadas       |
| Frecuencia SCL medida    | 83.07 kHz        | 83.43 kHz        |

### Escaneo de direcciones

El escaneo realizado mediante `i2c.scan()` detectó la dirección:

`['0x3c']`

Este resultado coincidió con las capturas de Logic 2, en las que la dirección `0x3C` recibió ACK, mientras que las direcciones no reconocidas recibieron NACK.

### Comandos observados en la OLED

| Función                 | Código hexadecimal | Resultado                            |
| ----------------------- | ------------------ | ------------------------------------ |
| Apagar pantalla         | `0xAE`             | Pantalla apagada                     |
| Encender pantalla       | `0xAF`             | Pantalla encendida                   |
| Configurar contraste    | `0x81` + valor     | Ajuste del contraste                 |
| Invertir imagen         | `0xA7`             | Visualización invertida              |
| Restaurar imagen normal | `0xA6`             | Visualización normal                 |
| Enviar datos            | `0x40` + datos     | Escritura de información en pantalla |

### Medición de la frecuencia SCL

Con una frecuencia configurada de 100 kHz, se midieron períodos de aproximadamente 12.038 µs y 11.987 µs, equivalentes a 83.07 kHz y 83.43 kHz, respectivamente.

Para el menú OLED, configurado a 50 kHz, se midió un período de aproximadamente 21 µs, equivalente a 47.6 kHz.

Estas mediciones permitieron comparar la frecuencia configurada en MicroPython con el comportamiento real del bus observado en el analizador lógico.


## ✅ Conclusiones

* Se reconoció la estructura de las transacciones I²C mediante la identificación de START, dirección, bit R/W, ACK/NACK y STOP.
* Se comprobó experimentalmente la diferencia entre una dirección reconocida y una dirección no reconocida.
* Se verificó la utilidad del método `i2c.scan()` para identificar dispositivos presentes en el bus.
* Se relacionaron los códigos hexadecimales transmitidos con las funciones de control de la pantalla OLED SSD1306.
* Se comprobó que la frecuencia real de SCL puede diferir de la configurada en el programa, por lo que es importante realizar mediciones con el analizador lógico.

## 👨‍💻 Autores

* **Harol Felipe Riveros Sierra** — 1401660
* **Salomé Bohórquez Blanco** — 1401654

**Programa:** Ingeniería en Telecomunicaciones
**Asignatura:** Comunicaciones Digitales
**Docente:** Ing. José de Jesús Rugeles Uribe
**Periodo académico:** 2026-2
