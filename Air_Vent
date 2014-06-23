/**********************************************
AIR VENT CONTROL
by Bithead942
 
 Loops continuously waiting for a network connection.
 If a network connection is detected (TCP 8888), check the temp of habitat and retun the value.
 
 Uses:
 -  Arduino Uno with ATmega328
 -  Arduino Ethernet Shield
 -  Up to 8 Servos to control air vents
****************************************************/
#include <Ethernet.h>
#include <SPI.h>
#include <Servo.h> 

const int VENT_OPEN = 70;
const int VENT_HALF = 115;
const int VENT_CLOSE = 180;

byte mac[] = { 
  0x??, 0xAD, 0xBE, 0x??, 0xFA, 0x?? };      // Replace ?? with hex
byte ip[] = { 
  192, 168, ???, ??? };                     //  Replace ??? with numeric
EthernetServer server(8888);

Servo Vent_Control[6];  // create servo object array to control 8 servos 
int Vent_State[6];      // create integer array to track state of 8 servos
int Vent_Pin[6] = {2, 3, 4, 5, 6, 7};        // create integer array to track control pin for 6 servos;  Pins 4, 10-13 used by Ethernet Shield
int i;
int SwitchPin = 9;
int TempPin = 0;

void setup() {
  pinMode(SwitchPin, OUTPUT);
  pinMode(TempPin, INPUT);

  //Intiialize all vents
  digitalWrite(SwitchPin, HIGH);
  for (i=0; i<=5; i++) {
    pinMode(Vent_Pin[i], OUTPUT);
    Vent_Control[i].attach(i+2);
    Vent_Control[i].write(VENT_CLOSE);
    delay(2500);
    Vent_State[i] = VENT_CLOSE;
  } 
  digitalWrite(SwitchPin, LOW);

  Ethernet.begin(mac, ip);
  server.begin();
}

void loop() {
  EthernetClient client = server.available();
  if (client) {
    while (client.connected()) {
      while (client.available() <= 0);                   //Wait for character
      char servo_id = client.read();
      if (servo_id >= 48 and servo_id <= 54) {           //Recevied a valid Servo ID (0 - 5) or Temp ID (6)
        if (servo_id == 54) {                            //Show Temp
           server.println(CheckTemp());
        }
        else {
           char servo_cmd = client.read();
           if (servo_cmd >= 48 and servo_cmd <= 50) {    //Received a valid Servo Command (0, 1, or 2)
             digitalWrite(SwitchPin, HIGH);
             if (servo_cmd == 48 and Vent_State[servo_id -48] != VENT_CLOSE) {
               Vent_Control[servo_id-48].write(VENT_CLOSE);
               Vent_State[servo_id-48] = VENT_CLOSE;
               server.println(0);
               delay(2500);
             }
             else if (servo_cmd == 49 and Vent_State[servo_id-48] != VENT_HALF) {
               Vent_Control[servo_id-48].write(VENT_HALF);
               Vent_State[servo_id-48] = VENT_HALF;
               server.println(0);
               delay(2500);
             }
             else if (servo_cmd == 50 and Vent_State[servo_id-48] != VENT_OPEN) {
               Vent_Control[servo_id-48].write(VENT_OPEN);
               Vent_State[servo_id-48] = VENT_OPEN;
               server.println(0);
               delay(2500);
             }
             else {
               server.println(3);                    //Vent already in requested state
             }
             digitalWrite(SwitchPin, LOW);

             }
          else {
            server.println(2);                       //Invalid Servo Command
          }
        }
      }
      else {
        server.println(1);                           //Invalid Servo ID
      }
    delay(2);
    client.flush();
    client.stop();
    }
  }
}   

double sampleToFahrenheit(uint16_t sample) {
  // conversion ratio in DEGREES/STEP:
  // (5000 mV / 1024 steps) * (1 degree / 10mV)
  //	^^^^^^^^^^^		 ^^^^^^^^^^
  //     from ADC		  from LM34
  return sample * (5000.0 / 1024.0 / 10.0);  
}


//unsigned int CheckTemp() {
  float CheckTemp() {
  unsigned int last_sample = 0;
  double this_temp = 0.0;
  double temp_avg = 0.0;
  int i;
  
    for(i=0; i<100; i++) {
      last_sample = analogRead(TempPin);
      this_temp = sampleToFahrenheit(last_sample);

      // add this contribution to the average
      temp_avg = temp_avg + this_temp;
    }
    temp_avg = temp_avg/100.0;
    
    return temp_avg;
}

