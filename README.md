# RDA-OLED-Potentiometer

This project implements an FM radio frequency scanner and volume control using the RDA5807 FM module, SSD1306 OLED display, and two potentiometers. It provides ready-to-run examples for both STM32 (CubeIDE/CubeMX) and Arduino platforms.

---

## Features

* FM frequency tuning across the 87.5 – 108.0 MHz band
* Real-time display of RSSI (signal strength) and FM\_TRUE status
* Volume adjustment via a potentiometer (0–15 scale)
* OLED shows current frequency, volume, and RSSI values
* Raw ADC readings sent to serial monitor (UART on STM32, Serial on Arduino)

---

## Hardware Connections

| Component                 | Arduino Pin  | STM32 Pin (CubeMX) |
| ------------------------- | ------------ | ------------------ |
| Potentiometer (frequency) |  A0          | PA0 (ADC\_IN0)     |
| Potentiometer (volume)    |  A1          | PA1 (ADC\_IN1)     |
| OLED SCL                  | A5 (I2C SCL) | PB6 (I2C1\_SCL)    |
| OLED SDA                  | A4 (I2C SDA) | PB7 (I2C1\_SDA)    |
| RDA5807 SCL               | A5 (I2C SCL) | PB6 (I2C1\_SCL)    |
| RDA5807 SDA               | A4 (I2C SDA) | PB7 (I2C1\_SDA)    |
| RDA5807 VCC               | 3.3 V        | 3.3 V              |
| RDA5807 GND               | GND          | GND                |

---

## Platform-Specific Setup & Usage

### STM32 (CubeIDE / CubeMX)

1. Open your `.ioc` file in CubeMX and enable:

   * **I2C1** on PB6/PB7
   * **ADC1** channels PA0 and PA1
   * **USART1** (or USART2) in Asynchronous mode on your chosen TX/RX pins
2. Click **Project ▶ Generate Code** to update the HAL drivers.
3. Verify that `Drivers/STM32L0xx_HAL_Driver/Inc/` contains the headers `stm32l0xx_hal_uart.h` and `stm32l0xx_hal_usart.h`.
4. In `main.c`, ensure your includes and extern declarations match:

   ```c
   #include "main.h"
   #include "stm32l0xx_hal.h"
   #include "stm32l0xx_hal_uart.h"
   #include "ssd1306.h"
   #include "ssd1306_fonts.h"
   #include "RDA_5807.h"
   #include "usart.h"

   extern ADC_HandleTypeDef hadc;
   extern UART_HandleTypeDef huart1; // or huart2
   extern RDA_HandleTypeDef handle;
   ```
5. Build and flash the project to your STM32 device.
6. Open a serial terminal at **115200 bps, 8N1** to view raw ADC logs.

### Arduino (IDE)

1. Launch Arduino IDE and install these libraries via Library Manager or GitHub:

   * **RDA5807** driver (e.g. from huddly/RDA5807)
   * **Adafruit SSD1306**
   * **Adafruit GFX**
2. Use the example sketch below (adjust pins if needed):

   ```cpp
   #include <Wire.h>
   #include <Adafruit_GFX.h>
   #include <Adafruit_SSD1306.h>
   #include <RDA5807.h>

   #define SCREEN_WIDTH 128
   #define SCREEN_HEIGHT 64
   Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);
   RDA5807 radio;

   void setup() {
     Serial.begin(115200);
     Wire.begin();
     display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
     radio.init();
   }

   void loop() {
     int rawFreq = analogRead(A0);
     int rawVol  = analogRead(A1);
     float freq  = map(rawFreq, 0, 1023, 875, 1080) / 10.0;
     int vol      = map(rawVol, 0, 1023, 0, 15);

     radio.setFrequency(freq);
     radio.setVolume(vol);
     int rssi = radio.getSignalLevel();

     display.clearDisplay();
     display.setTextSize(1);
     display.setTextColor(SSD1306_WHITE);
     display.setCursor(0, 0);
     display.print("Freq: "); display.print(freq); display.println(" MHz");
     display.print("Vol:  "); display.print(vol);
     display.print("  RSSI: "); display.print(rssi);
     display.display();

     Serial.printf("A0=%d A1=%d RSSI=%d\n", rawFreq, rawVol, rssi);
     delay(100);
   }
   ```
3. Upload to your Arduino board and open the Serial Monitor at **115200 baud**.

---

## License

This project is released under the **MIT License**. See `LICENSE` for details.
