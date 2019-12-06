# Diplomna_Rabota_2019-20_Petar_Ivanov
In this branch I will upload the code I use along the way.



This is the first code I programmed for this project.
  Blink

  Turns an LED on for one second, then off for one second, repeatedly.



const int LED = 20; 
 
// the setup function runs once when you press reset or power the board
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED, OUTPUT);
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(LED, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                       // wait for a second
}

So I used this on nrf51822 with motherboard with ST-Link V2 programmer. I programmed pin 20 of the motherboard to blink. 
This way I tested  how accessible is the programming of the motherboard using the arduino ide.

This is the next program.


// Import libraries (BLEPeripheral depends on SPI)
#include <SPI.h>
#include <BLEPeripheral.h>

// define pins (varies per shield/board)
#define BLE_REQ   10
#define BLE_RDY   2
#define BLE_RST   9

// LED pin
#define LED_PIN   20

// create peripheral instance, see pinouts above
BLEPeripheral            blePeripheral        = BLEPeripheral(BLE_REQ, BLE_RDY, BLE_RST);

// create service
BLEService               ledService           = BLEService("19b10000e8f2537e4f6cd104768a1214");

// create switch characteristic
BLECharCharacteristic    switchCharacteristic = BLECharCharacteristic("19b10001e8f2537e4f6cd104768a1214", BLERead | BLEWrite);

void setup() {
  Serial.begin(9600);
#if defined (__AVR_ATmega32U4__)
  delay(5000);  //5 seconds delay for enabling to see the start up comments on the serial board
#endif

  // set LED pin to output mode
  pinMode(LED_PIN, OUTPUT);

  // set advertised local name and service UUID
  blePeripheral.setLocalName("LED");
  blePeripheral.setAdvertisedServiceUuid(ledService.uuid());

  // add service and characteristic
  blePeripheral.addAttribute(ledService);
  blePeripheral.addAttribute(switchCharacteristic);

  // begin initialization
  blePeripheral.begin();

  Serial.println(F("BLE LED Peripheral"));
}

void loop() {
  BLECentral central = blePeripheral.central();

  if (central) {
    // central connected to peripheral
    Serial.print(F("Connected to central: "));
    Serial.println(central.address());

    while (central.connected()) {
      // central still connected to peripheral
      if (switchCharacteristic.written()) {
        // central wrote new value to characteristic, update LED
        if (switchCharacteristic.value()) {
          Serial.println(F("LED on"));
          digitalWrite(LED_PIN, HIGH);
        } else {
          Serial.println(F("LED off"));
          digitalWrite(LED_PIN, LOW);
        }
      }
    }

    // central disconnected
    Serial.print(F("Disconnected from central: "));
    Serial.println(central.address());
  }
}

So here we are using the BLEPeripheral library to programm the nrf51822.
This code allows us to remotely control the led pin 20 of the motherboard, using a phone.
We can send information from the phone using bluetooth to the module.

