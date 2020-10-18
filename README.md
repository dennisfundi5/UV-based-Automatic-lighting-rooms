# UV-based-Automatic-lighting-rooms

#include <LiquidCrystal_I2C.h> // Library for LCD

// Set the LCD I2C address
LiquidCrystal_I2C lcd = LiquidCrystal_I2C(0x27,16,2);

//Hardware pin definitions
int UVOUT = A0; //Output from the sensor
int REF_3V3 = A1; //3.3V power on the Arduino board
int lights = 9;

 
void setup()
{
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();
  lcd.begin(16,2);
 
  pinMode(UVOUT, INPUT);
  pinMode(REF_3V3, INPUT);
  // Set RelayPin as an output pin
  pinMode(lights, OUTPUT);
  
 lcd.setCursor(0,0);
 lcd.print("UV ROOM LIGHTING");
 Serial.println("UV ROOM LIGHTING");
 delay(4000);
  
}
 
void loop()
{
  int uvLevel = averageAnalogRead(UVOUT);
  int refLevel = averageAnalogRead(REF_3V3);
  
  //Use the 3.3V power pin as a reference to get a very accurate output value from sensor
  float outputVoltage = 3.3 / refLevel * uvLevel;
  
  float uvIntensity = mapfloat(outputVoltage, 0.99, 2.8, 0.0, 15.0); //Convert the voltage to a UV intensity level
 float IntensityF = uvIntensity*-10;
  Serial.print("output: ");
  Serial.print(refLevel);
  
 
  Serial.print("ML8511 output: ");
  Serial.print(uvLevel);
 
  Serial.print(" / ML8511 voltage: ");
  Serial.print(outputVoltage);
 
  Serial.print(" / UV Intensity (mW/cm^2): ");
  Serial.print(uvIntensity);
  Serial.print("  / IntensityF; ");
  Serial.print(IntensityF);
  //lcd.clear();
  //lcd.setCursor(0,0);
  //lcd.print("UV Ray Intensity");
  //lcd.setCursor(0, 1);
  //lcd.print(uvIntensity);
  //lcd.print(" mW/cm^2");
  Serial.println();
  delay(2000);
  
  if(IntensityF<21.00)
  {
    digitalWrite(lights, LOW);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Lights OFF");
    lcd.setCursor(0,1);
    lcd.print("nobody in room ");
    Serial.print("nobody in room\n");
    delay(10000);
  }
  else{
    digitalWrite(lights,HIGH);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Lights ON");
    lcd.setCursor(0,1);
    lcd.print("Someone  in room");
    Serial.print("someone  in room\n ");
    
    delay(4000);
    
  }}

 
//Takes an average of readings on a given pin
//Returns the average
int averageAnalogRead(int pinToRead)
{
  byte numberOfReadings = 8;
  unsigned int runningValue = 0; 
 
  for(int x = 0 ; x < numberOfReadings ; x++)
    runningValue += analogRead(pinToRead);
  runningValue /= numberOfReadings;
 
  return(runningValue);
}
 
float mapfloat(float x, float in_min, float in_max, float out_min, float out_max)
{
  return (x - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
}
