<!--

author:   Sebastian Zug; André Dietrich

email:    LiaScript@web.de

language: de

comment:  Dies ist ein interaktiver Demo-Kurs zu MicroPython basierend auf der MicroBit v2 Plattform. 

edit:    https://liascript.github.io/LiveEditor/?/show/file/https://raw.githubusercontent.com/LiaPlayground/microbit-demo-tutorial/refs/heads/main/README.md

import:  https://raw.githubusercontent.com/liaTemplates/webserial/main/README.md
         https://raw.githubusercontent.com/liaTemplates/MicroBit-Simulator/main/README.md

persistent: true

-->

[![LiaScript](https://raw.githubusercontent.com/LiaScript/LiaScript/master/badges/course.svg)](https://liascript.github.io/course/?https://raw.githubusercontent.com/LiaPlayground/microbit-demo-tutorial/refs/heads/main/README.md)
[![LiveEdit](https://raw.githubusercontent.com/LiaScript/LiaScript/refs/heads/development/badges/editor.svg)](https://liascript.github.io/LiveEditor/?/show/file/https://raw.githubusercontent.com/LiaPlayground/microbit-demo-tutorial/refs/heads/main/README.md)

# MicroPython auf dem BBC micro:bit

## 1. BBC micro:bit

Der BBC micro:bit ist ein kostengünstiges, programmierbares Board, entwickelt für Bildungszwecke:

* __Prozessor:__ 32-bit ARM Cortex-M0 (Nordic nRF51822)
* __Sensoren:__ Beschleunigungssensor, Kompass (Magnetometer), Temperatur
* __LED-Matrix:__ 5×5 LEDs
* __Buttons:__ A und B
* __Kommunikation:__ Bluetooth Low Energy, I²C, SPI, UART
* __Stromversorgung:__ USB oder Batteriehalter für 2 × AAA

??[Micro:Bit](https://sketchfab.com/3d-models/microbit-b453f11ad77a4545a33b3e0ecfba6fc5)

> **Anwendungsbeispiele**: Anzeige von Texten über LEDs, einfache Spiele, Sensor-Daten-Visualisierung, Bluetooth-Projekte.



## 2. Einführung in MicroPython

MicroPython ist eine schlanke Python-Implementierung für Mikrocontroller. Es ermöglicht dir, den micro\:bit mit einfachem Python-Code zu steuern.

* __Vorteile__:

  * Leichtgewichtig und ressourcenschonend
  * Gut dokumentiert und ideal für Einsteiger
  * Direkte Interaktion über die REPL (Konsole)

``` python
print(12 * 11)
```
@microbit

## 3. Dein erstes MicroPython-Programm

```python
from microbit import *

while True:
    display.scroll('Hallo!')
    sleep(1000)
```
@microbit

* **Erklärung**:

  * `from microbit import *`: Importiert alle Funktionen der micro\:bit-Bibliothek
  * `display.scroll('Hallo!')`: Scrollt den Text über die LEDs
  * `sleep(1000)`: Wartet 1000 ms (1 Sekunde)

---

## 4. Nächste Schritte

* Experimentiere mit den Sensoren:

  ```python
  from microbit import *
  while True:
      x = accelerometer.get_x()
      display.show(str(x))
      sleep(200)
  ```
  @microbit

* Nutze Tasten-Ereignisse:

  ```python
  from microbit import *

  while True:
      if button_a.was_pressed():
          display.show('A')
      if button_b.was_pressed():
          display.show('B')
  ```
  @microbit

Viel Spaß beim Programmieren mit deinem micro\:bit und MicroPython! 🎉


## 5. Live Programming

``` python
from microbit import *

# Display a scrolling message
display.scroll("Hello edrys!")

# Read the temperature
temp = temperature()
print("Temperature:", temp)

# Display a heart on the LED matrix
display.show(Image.HEART)
```
@WebSerial
