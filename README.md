# SPRUCE-A-GAMER-S-WEAR
sportsperson's wear

#include <SPI.h>
#include <SD.h>
#include "Wire.h" // This library allows you to communicate with I2C devices.
 File myFile;
const int MPU_ADDR = 0x68; // I2C address of the MPU-6050. If AD0 pin is set to HIGH, the I2C address will be 0x69.
int16_t accelerometer_x, accelerometer_y, accelerometer_z; // variables for accelerometer raw data
int16_t gyro_x, gyro_y, gyro_z; // variables for gyro raw data
char tmp_str[7]; // temporary variable used in convert function
char* convert_int16_to_str(int16_t i) { // converts int16 to string. Moreover, resulting strings will have the same length in the debug monitor.
  sprintf(tmp_str, "%6d", i);
  return tmp_str;
  
}
const int xpin=A2;  // Analog output pin to which xpin is connected
int ypin=A3;// Analog output pin to which ypin is connected 
int zpin=A6;// Analog output pin to which zpin is connected 
int powerpin=A0;// Analog output pin to which powerpin is connected 
int gnd=A1; // Analog output pin to which gnd is connected 

void setup()
{ 
  
  Serial.begin(9600);// initialize serial communications at 9600 bps:
  Wire.begin();
  Wire.beginTransmission(MPU_ADDR); // Begins a transmission to the I2C slave (GY-521 board)
  Wire.write(0x6B); // PWR_MGMT_1 register
  Wire.write(0); // set to zero (wakes up the MPU-6050)
  Wire.endTransmission(true);
pinMode(powerpin,OUTPUT);// initialize analog pin powerpin as an output.
pinMode(gnd,OUTPUT);// initialize analog pin gnd as an output.
digitalWrite(powerpin,HIGH);// set voltage high to powerpin
digitalWrite(gnd,LOW);//set voltage low to gnd 
while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

}
void loop()
{
  Wire.beginTransmission(MPU_ADDR);
  Wire.write(0x3B); // starting with register 0x3B (ACCEL_XOUT_H) [MPU-6000 and MPU-6050 Register Map and Descriptions Revision 4.2, p.40]
  Wire.endTransmission(false); // the parameter indicates that the Arduino will send a restart. As a result, the connection is kept active.
  Wire.requestFrom(MPU_ADDR, 7*2, true); // request a total of 7*2=14 registers
  float totvect[100]={0};
  float xaccl[100]={0};
  float yaccl[100]={0};
  float zaccl[100]={0};
  Serial.print("Initializing SD card...");
 if (!SD.begin(4)) {
//    Serial.println("initialization failed!");
//    return;
//  }
  Serial.println("initialization done.");
  for (int i=0;i<5;i++)//Use loop to run the commands continuosly
  {  // print the results to the Serial Monitor:
//Serial.println();
xaccl[i]=float(analogRead(xpin));
//Serial.println("Accelerometer: ");
//Serial.print("X = ");
//Serial.print(analogRead(xpin));

//delay(1);// wait for a microsecond

yaccl[i]=float(analogRead(ypin));
///ay(1);// wait for a microsecond

zaccl[i]=float(analogRead(zpin));
//Serial.print(" | Z = ");
//Serial.print(analogRead(zpin));
//delay(1);// wait for a microsecond
//Serial.println();
totvect[i] = sqrt(((xaccl[i]-xaccl[i-1])*(xaccl[i]-xaccl[i-1])) + ((yaccl[i]-yaccl[i-1])*(yaccl[i]-yaccl[i-1])) + ((zaccl[i]-zaccl[i-1])*(zaccl[i]-zaccl[i-1])));//formula to find velocity
//Serial.print("Velocity:");
//Serial.print(totvect[i]/100);//print the value of velocity
//Serial.print("m/s");//print unit of velocity
//delay(500);// wait for a half a second
  gyro_x = Wire.read()<<8 | Wire.read(); // reading registers: 0x43 (GYRO_XOUT_H) and 0x44 (GYRO_XOUT_L)
  gyro_y = Wire.read()<<8 | Wire.read(); // reading registers: 0x45 (GYRO_YOUT_H) and 0x46 (GYRO_YOUT_L)
  gyro_z = Wire.read()<<8 | Wire.read(); // reading registers: 0x47 (GYRO_ZOUT_H) and 0x48 (GYRO_ZOUT_L)
   // print the results to the Serial Monitor:
// Serial.println();
// Serial.println("Gyroscope: ");
// Serial.print(" | X = "); Serial.print(convert_int16_to_str(gyro_x));
// Serial.print(" | Y = "); Serial.print(convert_int16_to_str(gyro_y));
// Serial.print(" | Z = "); Serial.print(convert_int16_to_str(gyro_z));
//  Serial.println();
  
  // delay
  delay(500);// wait for a half a second

myFile = SD.open("test.txt", FILE_WRITE);
  if (myFile) {
  
    Serial.print("Writing to test.txt...");
    myFile.print("| Accelerometer: "); 
     myFile.print("| X = ");  myFile.print(analogRead(xpin));
   myFile.print(" | Y = ");  myFile.print(analogRead(ypin));
   myFile.print(" | Z = ");  myFile.print(analogRead(zpin));
    Serial.println();
      myFile.print(" | Velocity = ");  myFile.print(totvect[i]/100);
      myFile.print("m/s");
  // the following equation was taken from the documentation [MPU-6000/MPU-6050 Register Map and Description, p.30]
 // Serial.print(" | tmp = "); Serial.print(temperature/340.00+36.53);
 myFile.print("| Gyroscope: "); 
  myFile.print(" | X = ");  myFile.print(convert_int16_to_str(gyro_x));
  myFile.print(" | Y = ");  myFile.print(convert_int16_to_str(gyro_y));
  myFile.print(" | Z = ");  myFile.print(convert_int16_to_str(gyro_z));
  Serial.println();
  
  delay(5000);
   
    myFile.close();
    Serial.println("done.");
  } else {
    // if the file didn't open, print an error:
    Serial.println("error opening test.txt");
  }

  // re-open the file for reading:
  myFile = SD.open("test.txt");
  if (myFile) {
    Serial.println("test.txt:");

    // read from the file until there's nothing else in it:
    while (myFile.available()) {
      Serial.write(myFile.read());
    }
    // close the file:
    myFile.close();
  } else {
    // if the file didn't open, print an error:
    Serial.println("error opening test.txt");
  }
  }
}
}
