// Include all libraries
#include <SoftwareSerial.h> //library for GSM communication
#include <dht.h> //library for DHT11 sensor     

//Connections

//........GSM...........
SoftwareSerial SIM900A(2,3); // GSM TX to pin 2 and GSM RX to pin 3

//........PIR............
int calibrationTime = 10; //time taken for PIR sensor to caliberate
long unsigned int lowIn;         
long unsigned int pause = 1000;  
boolean lockLow = true;
boolean takeLowTime;  
int pirPin = 4;

//......ULTRASONIC.......
const int trigPin = 10;
const int echoPin = 9;
long duration;
float distance;
int tankPumpSMSCount = 0; // number of times SMS sent

//......DHT11.......
#define outPin 11       // Defines pin number to which the sensor is connected
dht DHT;                // Creates a DHT object

//......SOIL_MOISTURE.......
const int sensor_pin = A1;	/* Soil moisture sensor O/P pin */
float moisture_percentage;
int sensor_analog;
int irrigationPumpSMSCount = 0;

void setup() {
  // put your setup code here, to run once:

//........GSM...........
SIM900A.begin(9600); // GSM Module Baud rate – communication speed
Serial.begin(9600); // Baud rate of Serial Monitor in the IDE app
Serial.println ("Text Messege Module Ready & Verified");
delay(100);
//........PIR............
pinMode(pirPin, INPUT);
digitalWrite(pirPin, LOW);
Serial.print("calibrating PIR sensor ");
for(int i = 0; i < calibrationTime; i++){
  Serial.print(".");
}
Serial.println(" done PIR SENSOR ACTIVE");
//......ULTRASONIC.......
pinMode(trigPin, OUTPUT);
pinMode(echoPin, INPUT);
pinMode(8,OUTPUT);

//......SOIL_MOISTURE.......
pinMode(5,OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:

//......SOIL_MOISTURE.......
  sensor_analog = analogRead(sensor_pin);
  moisture_percentage = ( 100 - ( (sensor_analog/1023.00) * 100 ) );
  Serial.print("  Moisture Percentage = ");
  Serial.print(moisture_percentage);
  Serial.print("%\n\n");
  if (moisture_percentage>50.00){
  irrigationPumpSMSCount = 0;
  Serial.println("Moisture level: Sufficient");
  digitalWrite(5,LOW);
  delay(200);

  }
  else{
  Serial.println("Moisture level: Not Sufficient");
  digitalWrite(5,HIGH);
  while(irrigationPumpSMSCount<2){
      Serial.println ("Sending Message please wait….");
      SIM900A.println("AT+CMGF=1");
      delay(1000);
      SIM900A.println("AT+CMGS=\"+918348847566\"\r");
      delay(1000);
      SIM900A.println("Moisture level not sufficient. Irrigation Pump ON.");// Messsage content
      delay(1000);
      SIM900A.println((char)26);//GSM module recognises end of message content
      delay(1000);
      Serial.println ("Message sent succesfully");
      irrigationPumpSMSCount++;
  }
  }

//......ULTRASONIC.......
  // Clears the trigPin
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  // Sets the trigPin on HIGH state for 10 micro seconds
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  // Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(echoPin, HIGH);
  distance = duration * 0.034 / 2;
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm ");

  if(distance>=4){
    digitalWrite(8,HIGH); //if water level distance is more than 25cm, relay and pump will turn ON
    Serial.println ("tank pump on ");
    while(tankPumpSMSCount<2){
      Serial.println ("Sending Message please wait….");
      SIM900A.println("AT+CMGF=1");
      delay(1000);
      SIM900A.println("AT+CMGS=\"+918348847566\"\r");
      delay(1000);
      SIM900A.println(" Tank is low on water level. Tank Pump ON.");// Messsage content
      delay(1000);
      SIM900A.println((char)26);//GSM module recognises end of message content
      delay(1000);
      Serial.println ("Message sent succesfully");
      tankPumpSMSCount++;
  }
}
else{
  tankPumpSMSCount = 0;
  digitalWrite(8,LOW); //if water level distance is lesser than 25cm, relay and pump will turn OFF
}

//........PIR............
  if(digitalRead(pirPin) == HIGH){
  Serial.println ("Motion Detected");
  Serial.println ("Sending Message please wait….");
  SIM900A.println("AT+CMGF=1");
  delay(1000);
  SIM900A.println("AT+CMGS=\"+918348847566\"\r");
  delay(1000);
  SIM900A.println("Intruder Alert!");// Messsage content
  delay(1000);
  SIM900A.println((char)26);//GSM module recognises end of message content
  delay(1000);
  Serial.println ("Message sent succesfully");
}

 //......DHT11.......
  int readData = DHT.read11(outPin);
	float t = DHT.temperature;        // Read temperature
	float h = DHT.humidity;           // Read humidity

	Serial.print("Temperature = ");
	Serial.print(t);
	Serial.print("°C | ");
	Serial.print((t*9.0)/5.0+32.0);        // Convert celsius to fahrenheit
	Serial.println("°F ");
	Serial.print("Humidity = ");
	Serial.print(h);
	Serial.println("% ");
	Serial.println("");
}
