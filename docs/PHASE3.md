# Phase 3: Panda Vision
The core of this phase is the transition from a local hardware setup to a distributed AI architecture / IoT system. By using the Pico 2 WH, you've ensured that the visual display (the "Kitchen Node") is independent of the LattePanda's (the "Brain") processing load.

## Hardware Components
The Brain (LP3D) identifies a specific user through the camera feed using the AI accelerator. It sends a wireless command (the "Communication Protocol") over the local network to the Messenger (Pico 2 WH).  The Messenger receives this command and instantly renders the appropriate "Welcome Home" message on the Adafruit LED Matrix.
>  *Detection > Network Trigger > Visual Output*

### The Brain: Inference & Logic (LattePanda)
* **Role:** Runs the AI models (YOLOv8/v9) to recognize "Nathan" and sends a network command.
* **Key Advantage:** The 25 TOPS from the DX-M1 handles the high-resolution camera feed without breaking a sweat, leaving the CPU free to manage the network bridge.
* **Communication Protocol (MQTT):** Ensuring near-zero latency between face detection and the visual display.
  - **Broker:** Running locally on the LattePanda
  - **Topic:** `panda/vision/alerts`
  - **Payload Format (JSON):**
    ```json
    {
      "event": "recognition",
      "subject": "Nathan",
      "message": "Welcome Home Nathan!",
      "color": [0, 255, 0]
    }
    ```
  - **Pico Logic:** The Pico 2 WH subscribes to this topic. Upon receipt, it parses the JSON and triggers the corresponding animation on the Adafruit LED Matrix.

### The Messenger (Pico 2 WH)
* **Role:** Stays in the kitchen, receives Wi-Fi signals and drives the LED Matrix.
* **Key Advantage:** The Pico 2 WH features the new RP2350 chip with 520KB of RAM, which is more than enough to buffer complex scrolling text and high-speed animations for the LED matrix.
**

### The Visuals (Adafruit LED Matrix)
* **Matrix:** The Adafruit 16x32 RGB Matrix provides high-impact visual feedback that is clearly visible from across the room.
* **Signal Integrity:** The 74AHCT125 Level Shifter is essential for converting the Pico's 3.3V signals to the 5V signals required by the matrix, preventing flickering or "ghosting."
* **Stable Power:** The 5V 4A Power Supply ensures that even if you turn all 512 LEDs to full brightness white, the system remains stable.
* **Visuals & Connectivity Hardware:**
  - **Micro-controller:** [Raspberry Pi Pico 2 WH](https://www.raspberrypi.com/products/raspberry-pi-pico-2/) (Dual-core RP2350, 520KB RAM, Wi-Fi)
  - **LED Matrix:** [Adafruit Industries Medium 16x32 RGB LED Matrix Panel](https://www.microcenter.com/product/656404/adafruit-industries-medium-16x32-rgb-led-matrix-panel)
  - **Level Shifter:** [74AHCT125 Quad Level-Shifter](https://www.adafruit.com/product/1787) (3.3V to 5V signal conversion)
  - **Power Supply:** [5V 4A Switching Power Supply](https://www.microcenter.com/product/456255/adafruit-industries-5v-4a-4000ma-switching-power-supply) (Powers both Matrix and Pico)
  - **Protection:** [1N5817 Schottky Diode](https://stompboxparts.com/semiconductors/1n5817-schottky-diode/) (Prevents back-powering USB during programming)
  - **Connector:** [DC Barrel Jack to 2-pin Terminal Block Adapter](https://www.pololu.com/product/2449)

> *Pro-Tip for Assembly: When you wire the 74AHCT125, ensure you tie the OE (Output Enable) pins to Ground. This keeps the outputs active at all times, ensuring your matrix doesn't go dark if the Pico 2 WH momentarily shifts its pin states during a reboot.*

---