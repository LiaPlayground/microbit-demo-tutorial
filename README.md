<!--

edit:   https://liascript.github.io/LiveEditor/?/show/file/https://raw.githubusercontent.com/LiaPlayground/microbit-demo-tutorial/refs/heads/main/README.md

import: https://raw.githubusercontent.com/liaTemplates/webserial/main/README.md

@microbit: @microbit_(@uid,```@input```)

@microbit_
<script>
const iframe = document.querySelector('#simulator_@0')
const simulator = iframe.contentWindow

const onMessage = (e) => {
    if (e.source !== simulator) return;
    const { data } = e;
    switch (data.kind) {
      case 'serial_output':
        console.stream(data.data);
        break;
      case 'log_output':
        if (data.headings) console.log(data.headings);
        if (data.data)     console.log(data.data);
        break;
      case 'log_delete':
        console.debug('[log_delete]');
        break;
      case 'request_flash':
        send.lia("LIA: terminal");
        simulator.postMessage({
          kind: 'flash',
          filesystem: {
            'main.py': new TextEncoder().encode(`@1`),
          },
        }, '*');
        break;
      case 'internal_error':
        console.error('Simulator internal error:', data.error);
        break;
    }
  };

window.addEventListener('message', onMessage);

send.handle("input", (data) => {
  simulator.postMessage({
      kind: 'serial_input',
      data: data + "\r\n",
    },
    '*'
  )
})

send.handle("stop", () => {
    // detach the message listener
    window.removeEventListener('message', onMessage);

    // reset the iframe by reloading its src
    // (you can also set src="" then back, or completely remove/add the element)
    iframe.src = iframe.src;

    // optionally hide it if you want:
    iframe.style.display = "none";
  });

iframe.style.display = "block"

"LIA: wait"
</script>

<iframe
  id="simulator_@0"
  src="https://python-simulator.usermbit.org/v/0.1/simulator.html?color=red"
  title="Simulator"
  frameborder="0"
  scrolling="no"
  sandbox="allow-scripts allow-same-origin"
  style="width: 100%; max-width: 500px; aspect-ratio: 10 / 8; display: none"
></iframe>
@end

persistent: true

-->

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
