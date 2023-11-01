# FlowSensor-Arduino-NodeMCU
 //Arduino IDE Code for taking the data readings  from mutiple flow sensors and sending the sensor data to the NodeMCU through serial communication.



volatile int flow_frequency2; // Measures flow sensor pulses
volatile int flow_frequency3;
// Calculated litres/hour
double diff;
 double vol = 0.0,l_minute2,l_minute3;
unsigned char flowsensor2 = 2; //sensor input2
unsigned char flowsensor3 = 3; // Sensor Input3

unsigned long currentTime;

unsigned long cloopTime;




void flow2() // Interrupt function

{

   flow_frequency2++;

}
void flow3(){

   flow_frequency3++;
}




  

  



//Arduino to NodeMCU Lib
#include <SoftwareSerial.h>
#include <ArduinoJson.h>

//Initialise Arduino to NodeMCU (5=Rx & 6=Tx)
SoftwareSerial nodemcu(5, 6);



double  temp;


void setup() {
  Serial.begin(9600);

  nodemcu.begin(9600);
  
   pinMode(flowsensor2, INPUT);
      pinMode(flowsensor3, INPUT);

   digitalWrite(flowsensor2, HIGH); // Optional Internal Pull-Up
     digitalWrite(flowsensor3, HIGH);

 

   attachInterrupt(digitalPinToInterrupt(flowsensor2), flow2, RISING); 
      attachInterrupt(digitalPinToInterrupt(flowsensor3), flow3, RISING);// Setup Interrupt

   

   currentTime = millis();

   cloopTime = currentTime;

  Serial.println("Program started");
  delay(1000);
}

void loop() {

  StaticJsonBuffer<1000> jsonBuffer;
  JsonObject& data = jsonBuffer.createObject();

  dht11_func();

 data["flowDifference"] = diff;
 data["flowrate1"]=l_minute2;
 data["flowrate2"]=l_minute3;
 
  data.printTo(nodemcu);
  jsonBuffer.clear();

  delay(1000);
}

void dht11_func() {
  

   currentTime = millis();

   // Every second, calculate and print litres/hour

   if(currentTime >= (cloopTime + 1000))

   {

   cloopTime = currentTime; // Updates cloopTime

   if(flow_frequency2 != 0){

      // Pulse frequency (Hz) = 7.5Q, Q is flow rate in L/min.

  l_minute2= (flow_frequency2/ 7.5); // (Pulse frequency x 60 min) / 7.5Q = flowrate in L/hour

   

  l_minute2= l_minute2/60;

    

  vol = vol +l_minute2;
       

  flow_frequency2= 0; // Reset Counter

  Serial.print(l_minute2, DEC); // Print litres/hour
      
  Serial.println(" L/Sec comes from 2 ");
      

 }
    if(flow_frequency3 != 0){

      // Pulse frequency (Hz) = 7.5Q, Q is flow rate in L/min.

  l_minute3= (flow_frequency3/ 7.5); // (Pulse frequency x 60 min) / 7.5Q = flowrate in L/hour

   

   l_minute3= l_minute3/60;

    

   vol = vol +l_minute3;

     
  flow_frequency3= 0; // Reset Counter

  Serial.print(l_minute3, DEC); // Print litres/hour
       Serial.println(" L/Sec comes from 3 ");
    
     
   diff=l_minute2-l_minute3;
      Serial.println(abs(diff));
    }
    
   else {
  
   diff=0.0;
    Serial.println(abs(diff));

   }
 delay(1000);

   }

}
