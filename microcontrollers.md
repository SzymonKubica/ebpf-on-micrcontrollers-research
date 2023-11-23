# Target Microcontroller

[Main Page](./README.md)

Currently we are considering the following alternatives:

- STM32F4O1RE [available here](https://www.digikey.co.uk/en/products/detail/stmicroelectronics/NUCLEO-F401RE/4695525?utm_adgroup=&utm_source=google&utm_medium=cpc&utm_campaign=PMax%20Shopping_Product_New%20Customer%20Acquisition&utm_term=&productid=4695525&utm_content=&utm_id=go_cmp-19905262708_adg-_ad-__dev-c_ext-_prd-4695525_sig-CjwKCAiAu9yqBhBmEiwAHTx5p0oDFLHI27qDMUszlzZ1zKjkvd_Kp9xLMpejBuPKUoI78xomgQiMpBoCUOwQAvD_BwE&gad_source=1&gclid=CjwKCAiAu9yqBhBmEiwAHTx5p0oDFLHI27qDMUszlzZ1zKjkvd_Kp9xLMpejBuPKUoI78xomgQiMpBoCUOwQAvD_BwE)
  - FreeRTOS has good support for it
  - No built-in wifi support (expansion boards needed -> possibly need another
    esp32 just for the wifi)
  - runs the ARM Cortex M4 cpu
- Arduino Uno Rev4 [available here](https://store.arduino.cc/pages/uno-r4)
  - No first-class support by FreeRTOS (not explicitly listed in the docs,
    however some ports of the RTOS exist for the old arduinos).
  - Integrated wifi support via an onboard esp32 (can be flashed with [custom
    firmware](https://docs.arduino.cc/tutorials/uno-r4-wifi/esp32-upload))
  - Came out very recently so some libraries might not have the correct HALs
    for the pinout
- esp32 (or compatible board [available here](https://www.amazon.co.uk/ESP-WROOM-32-Development-Dual-Mode-Microcontroller-Integrated/dp/B07YKBY53C/ref=asc_df_B07YKBY53C/?tag=googshopuk-21&linkCode=df0&hvadid=658814687030&hvpos=&hvnetw=g&hvrand=4700225218477974778&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=1006886&hvtargid=pla-860392095046&psc=1&mcid=8ebdb19acd703d809ea4a87eda3fba3c)
  - esp-idf has a great documentation and is easy to use
  - I already have 4 of those modules
  - the chip has an integrated wifi module and first class support
  - has two distinct cores so is more viable for the femto-containers-like approach
  - **disadvantage**: Runs on the Xtensa architecture so the existing uBPF or rBPF
    userspace VMs don't support it.


