#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 20, 4); // set the LCD address to 0x27 for a 16 chars and 2 line display

#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <String.h>
#define Step 15

//SSID and Password of your WiFi router
const char* ssid = "RedmiNote";
const char* password = "rana6963";
String setcolor = "#000000"; //Set color for HTML
String Color;
ESP8266WebServer server(80);

const char MAIN_page[] PROGMEM = R"=====(
<!DOCTYPE html>
<html>
<head>
<meta name='viewport' content='width=device-width, initial-scale=1.0'/>
<meta charset='utf-8'>
<meta http-equiv='refresh' content='1'>
</head>
<body style="background:@@color@@;">
<meta http-equiv=\"refresh\" content=\"1\">
</body>
</html>
)=====";
//=======================================================================
//                    handles main page
//=======================================================================
void handleRoot() {
  String p = MAIN_page;  
  setcolor = Color;
  p.replace("@@color@@",setcolor);    //Set page background color and selected color
  server.send(200, "text/html", p);    
}

const int s0 = D3;
const int s1 = D4;
const int s2 = D5;
const int s3 = D6;
const int out = D7;
byte red = 0;
byte green = 0;
byte blue = 0;


#define sample 15
byte RED_S[sample];
byte GREEN_S[sample];
byte BLUE_S[sample];
int i = 0;

byte AVR = 0;
byte AVG = 0;
byte AVB = 0;


void setup()
{
  lcd.init();                      // initialize the lcd
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Shayan");
  delay(1000);

  Serial.begin(9600);
  pinMode(s0, OUTPUT);
  pinMode(s1, OUTPUT);
  pinMode(s2, OUTPUT);
  pinMode(s3, OUTPUT);
  pinMode(out, INPUT);
  digitalWrite(s0, HIGH);
  digitalWrite(s1, HIGH);

  WiFi.begin(ssid, password);     //Connect to your WiFi router
  Serial.println("");

  // Wait for connection
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  //If connection successful show IP address in serial monitor
  Serial.println("");
  Serial.print("Connected to ");
  Serial.println(ssid);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());  //IP address assigned to your ESP

  server.on("/", handleRoot);  //Associate handler function to path

  server.begin();                           //Start server
  Serial.println("HTTP server started");
}
void loop()
{
  
  color();
  
  RED_S[i] = red;
  GREEN_S[i] = green;
  BLUE_S[i] = blue;
  i ++;
  if ( i == sample)
  {
    String r,g,b;
    AVR = Average(RED_S);
    AVG = Average(GREEN_S);
    AVB = Average(BLUE_S);
    r = String(AVR, HEX);
    g = String(AVG, HEX);
    b = String(AVB, HEX);
    
    
    Color = "#" + r + g + b  ;
    Serial.println(Color);
    server.handleClient();
    
    PROC();
    
    for (int j = 0 ; j < sample - 10 ; j++)
    {
      RED_S[j] = RED_S[j + 10];
      GREEN_S[j] = GREEN_S[j + 10];
      BLUE_S[j] = BLUE_S[j + 10];
    }
    i = sample - 10;
  }
  delay(200);
}
void color()
{
  //String RED , BLUE , GREEN ;
  digitalWrite(s2, LOW);
  digitalWrite(s3, LOW);
  //count OUT, pRed, RED
  red = pulseIn(out, digitalRead(out) == HIGH ? LOW : HIGH);
  digitalWrite(s3, HIGH);
  //count OUT, pBLUE, BLUE
  blue = pulseIn(out, digitalRead(out) == HIGH ? LOW : HIGH);
  digitalWrite(s2, HIGH);    //count OUT, pGreen, GREEN
  green = pulseIn(out, digitalRead(out) == HIGH ? LOW : HIGH);
  red = 245 - red;
  green = 242 - green;
  blue = 235 - blue;

}
int Average(byte a[sample])
{
  int sum = 0;
  for (int j = 0 ; j < sample ; j++)
  {
    sum += a[j];
  }
  return sum / sample;
}


void PROC()
{
  
  Serial.print("R Intensity:");
  Serial.print(AVR, DEC);
  Serial.print(" G Intensity: ");
  Serial.print(AVG, DEC);
  Serial.print(" B Intensity : ");
  Serial.print(AVB, DEC);
  Serial.println();
  
  lcd.backlight();
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(AVR, DEC);
  lcd.print(",");
  lcd.print(AVG, DEC);
  lcd.print(",");
  lcd.print(AVB, DEC);
  
  if( AVR > 190 - Step && AVR < 190 + Step && AVG > 100 - Step && AVG < 100 + Step && AVB > 135 - Step && AVB < 135 + Step)
  {
    lcd.setCursor(0, 1);
    lcd.print("RED");
  }
  else if( AVR > 85 - Step && AVR < 85 + Step && AVG > 100 - Step && AVG < 100 + Step && AVB > 100 - Step && AVB < 100 + Step)
  {
    lcd.setCursor(0, 1);
    lcd.print("GREEN");
  }
  else if( AVR > 90 - Step && AVR < 90 + Step && AVG > 150 - Step && AVG < 150 + Step && AVB > 180 - Step && AVB < 180 + Step)
  {
    lcd.setCursor(0, 1);
    lcd.print("BLUE");
  }
  else if( AVR > 185 - Step && AVR < 185 + Step && AVG > 145 - Step && AVG < 145 + Step && AVB > 100 - Step && AVB < 100 + Step)
  {
    lcd.setCursor(0, 1);
    lcd.print("YELLOW");
  }
}
