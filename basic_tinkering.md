## Day 1: Tinkering with the tools.

### Tools
- Arduino Uno
- Breadboard
- LED
- Buzzer
- Connecting wires

### Objective of the day
- Turn on the LED and buzzer, and turn off the built in LED.
- Wait for 1 second.
- Turn off the LED and buzzer, and turn on the built in LED.
- Wait for 1 second.
- Repeat.

### Circuit
<img src="basic_tinkering.svg" width=600>

### Code
```
const int LED = 8;
const int BUZZER = 10;

void setup()
{
    // changing required ports to output mode
    pinMode(LED, OUTPUT);
    pinMode(BUZZER, OUTPUT);
    // LED_BUILTIN is a pre-defined macro for builtin LED
    pinMode(LED_BUILTIN, OUTPUT);
}

void loop()
{
    digitalWrite(LED, HIGH);
    digitalWrite(BUZZER, HIGH);
    digitalWrite(LED_BUILTIN, LOW);
    delay(1000);                            // milliseconds
    digitalWrite(LED, LOW);
    digitalWrite(BUZZER, LOW);
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);
}
```