<!--

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


@WebSerial
<script>
(async function() {
  // Check if the Web Serial API is supported.
  if (!("serial" in navigator)) {
    setTimeout(() => {
        console.error("Web Serial API is not supported in this browser, try Chrome.");
    }, 100)
    send.lia("LIA: stop")
    return;
  }

  // Declare connection-related variables for later cleanup.
  let port = null;
  let reader = null;

  try {
    // Request and open the serial port.
    port = await navigator.serial.requestPort();
    await port.open({ baudRate: 115200 });

    // Create a TextEncoder instance.
    const encoder = new TextEncoder();
    // Function to stop any currently running code by sending Ctrl-C.
    async function stopCurrentProgram() {
      try {
        const writer = port.writable.getWriter();
        // Send Ctrl-C (ASCII 0x03) to interrupt any running code.
        await writer.write(encoder.encode("\x03"));
        // Wait briefly to allow the interrupt to be processed.
        await new Promise(resolve => setTimeout(resolve, 100));
        // Send a second Ctrl-C in case the first one was missed.
        await writer.write(encoder.encode("\x03"));
        writer.releaseLock();
      } catch (e) {
        console.error("Error sending Ctrl-C:", e);
      }
    }

    // Stop any running code before sending new code.
    await stopCurrentProgram();

    // Retrieve the entire Python code from the liascript input.
    const pythonCode = `@input(0)`;

    // Function to send code using MicroPython's paste mode.
    // In paste mode, the REPL buffers all lines until Ctrl‑D is received,
    // then it compiles and executes the entire code block at once.
    async function sendCodeInPasteMode(code) {
      const writer = port.writable.getWriter();
      // Enter paste mode (Ctrl‑E, ASCII 0x05).
      await writer.write(encoder.encode("\x05"));
      // Wait briefly for paste mode to be activated.
      await new Promise(resolve => setTimeout(resolve, 100));

      // Split the code into lines, preserving all indentation.
      const codeLines = code.split(/\r?\n/);
      for (const line of codeLines) {
        // Send each line exactly as-is, with CR+LF.
        await writer.write(encoder.encode(line + "\r\n"));
      }
      // Exit paste mode by sending Ctrl‑D (ASCII 0x04).
      await writer.write(encoder.encode("\x04"));
      writer.releaseLock();
      send.lia("LIA: terminal");
    }

    // Function that sends the code and reads output until the REPL prompt (">>>") is detected.
    // This ensures the entire block is executed before further input is allowed.
    async function sendCodeAndWaitForPrompt(code) {
      await sendCodeInPasteMode(code);
      let outputBuffer = "";
      const tempReader = port.readable.getReader();
      const decoder = new TextDecoder();
      let promptFound = false;

      while (!promptFound) {
        const { value, done } = await tempReader.read();
        if (done) break;
        if (value) {
          const text = decoder.decode(value);
          outputBuffer += text;
          console.stream(text);
          // Look for the REPL prompt (adjust if your prompt differs).
          if (outputBuffer.includes(">>>")) {
            promptFound = true;
          }
        }
      }
      await tempReader.releaseLock();
      return outputBuffer;
    }

    // Send the Python code and wait until the prompt is detected.
    await sendCodeAndWaitForPrompt(pythonCode);
    console.log("Python code executed and prompt detected.");

    // Now that execution is complete, enable terminal input.
    send.lia("LIA: terminal");

    // Start a global read loop to capture and display subsequent output.
    reader = port.readable.getReader();
    const globalDecoder = new TextDecoder();
    (async function readLoop() {
      try {
        while (true) {
          const { value, done } = await reader.read();
          if (done) {
            console.debug("Stream closed");
            send.lia("LIA: stop");
            break;
          }
          if (value) {
            console.stream(globalDecoder.decode(value));
          }
        }
      } catch (error) {
        console.error("Read error:", error);
      } finally {
        try { reader.releaseLock(); } catch (e) { /* ignore */ }
      }
    })();

    // Handler to send terminal input lines to MicroPython.
    send.handle("input", input => {
      (async function() {
        try {
          const writer = port.writable.getWriter();
          // Send the terminal input (preserving any whitespace) with CR+LF.
          await writer.write(encoder.encode(input + "\r\n"));
          writer.releaseLock();
        } catch (e) {
          console.error("Error sending input to MicroPython:", e);
        }
      })();
    });

    // Handler to clean up all connections and variables when a "stop" command is received.
    send.handle("stop", async () => {
      console.log("Cleaning up connections and stopping execution.");

      // Cancel the reader if it exists.
      if (reader) {
        try {
          await reader.cancel();
        } catch (e) {
          console.error("Error canceling reader:", e);
        }
        try { reader.releaseLock(); } catch (e) { /* ignore */ }
      }

      // Close the serial port if it's open.
      if (port) {
        try {
          await port.close();
        } catch (e) {
          console.error("Error closing port:", e);
        }
      }

      // Reset connection variables.
      port = null;
      reader = null;
      console.log("Cleanup complete.");
    });

  } catch (error) {
    console.error("Error connecting to the MicroPython device:", error);
    send.lia("LIA: stop");
  }
})();

"LIA: wait"
</script>
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
