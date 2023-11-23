# Target RTOS selection

[Main Page](./README.md)

The main options we can consider:

- FreeRTOS ([here](https://www.freertos.org/Documentation/RTOS_book.html))
  - commonly used
  - good support for stm32
- esp-idf ([here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/))
  - commonly used
  - great documentation and first class support with esp32
  - I have experience with it and the tooling is quite good
- Tock ([here](https://github.com/tock/tock/blob/master/doc/Overview.md))
  - written in rust
  - relies on the microcontroller having and MPU
  - not that many microcontrollers supported

