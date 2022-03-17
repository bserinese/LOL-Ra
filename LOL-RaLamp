/* Welcome to the LOL-Ra Lamp by Bryan Serinese
 * This project uses the Adafruit LoRa Feather to display an image on a 16x16 RGB matrix.  
 * The 8-way rotary selector allows you to pick 1 of 8 images to display and a potentiometer allows you to select the color.
 * A send button engages the onboard LoRa radio to transmit the image and color to the partner lamp, which then displays the image 
 * The partner lamp will continue to display the sent image until it sends its own image back.
 * You need to add the Adafruit boards to the Board Manager and use Adafruit Feather 32u4
 * I based the LoRa transmission code off of the LoRa example code:  
 * LoRa Duplex communication with callback by Tom Igoe created 28 April 2017 
 */
// include libraries
#include <avr/pgmspace.h>  // Needed to store image arrays in Flash using PROGMEM
#include "FastLED.h"       // Fastled library to control the LEDs
#include <SPI.h>             
#include <LoRa.h>         // LoRa library by Sandeep Mistry

#define NUM_LEDS 256  //16 X 16 LED matrix
CRGB leds[NUM_LEDS]; // Define the array of leds
#define DATA_PIN 2  // Define the Data Pin Connected to the data pin of the matrix

#define SendButton 3 // Define the Data Pin for the Send push button

#define ImageSwitch_1Pin 5 // Define the pins for the 8-way Rotary switch.  Once you wire it up, you'll have to figure out which position goes with which images and redefine them. I found that easier than trying to follow the wires
#define ImageSwitch_2Pin 6
#define ImageSwitch_3Pin 9
#define ImageSwitch_4Pin 10
#define ImageSwitch_5Pin 11 
#define ImageSwitch_6Pin 12
#define ImageSwitch_7Pin 13 
#define ImageSwitch_8Pin 22 //this pin is labeled A4 (analog 4) but it can also be addressed as digital 22

const int csPin = 8;          // LoRa radio chip select
const int resetPin = 4;       // LoRa radio reset
const int irqPin = 7;         // change for your board; must be a hardware interrupt pin

String outgoing;              // outgoing message
String message;               //used to concate the Image and Color into the outgoing message
byte msgCount = 0;            // count of outgoing messages
byte localAddress = 0xBF;     // address of this device
byte destination = 0xBF;      // destination to send to
long lastSendTime = 0;        // last send time
int interval = 2000;          // interval between sends

long ColorVal[10] = {  //this is the array that stores the different colors for the background. The potentiometer will map to resistance values to these positions
  0x000000,   //off
  0xFF0000,   //red
  0xFFC0CB,   //pink
  0xFFA500,   //Oranges
  0xFFFF00,  //yellow 
  0x0000FF,   //blue
  0x00FF00,  //greens
  0xE6E6FA,  //indigos
  0xEE82EE,  //violets
  0xFFFFFF    //white
                  };
int const ColorPoTPin = A0;  //connect the middle pin of the potentiometer to the analog 0 pin
int ColorPotVal = 0;
int ColorSpot;
int ImageSpot;
  
// Create the array of images for the 16X16 matrix and store it in Flash memory using PROGREM otherwise the Arduino memory cannot handle so many arrays

//The 1 represents a value that can be changed by the color selector. The rest are the hex codes for the RGB value that I want to be constant. In this case, the smiley face will always be yellow but the background can change
const long Smiley[256] PROGMEM =
{1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,
1,1,1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,1,1,
1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,
1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,
1,1,0xFFFF00,0xFFFF00,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,0xFFFF00,0xFFFF00,1,1,
1,1,0xFFFF00,0xFFFF00,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,0xFFFF00,0xFFFF00,1,1,
1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,
1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,
1,0xFFFF00,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,0xFFFF00,1,
1,0xFFFF00,0xFFFF00,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,0xFFFF00,0xFFFF00,1,
1,1,0xFFFF00,0xFFFF00,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,0xFFFF00,0xFFFF00,1,1,
1,1,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,1,1,
1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,
1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,
1,1,1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,1,1,
1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
};

const long Frowny[256] PROGMEM =
{1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,
1,1,1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,1,1,
1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,
1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,
1,1,0xFFFF00,0xFFFF00,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,0xFFFF00,0xFFFF00,1,1,
1,1,0xFFFF00,0xFFFF00,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,0xFFFF00,0xFFFF00,1,1,
1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,
1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,
1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,
1,0xFFFF00,0xFFFF00,0xFFFF00,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,0xFFFF00,0xFFFF00,0xFFFF00,1,
1,1,0xFFFF00,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,0xFFFF00,1,1,
1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,
1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,
1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,
1,1,1,1,1,1,0xFFFF00,0xFFFF00,0xFFFF00,0xFFFF00,1,1,1,1,1,1,
1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
};

// for the words, the background is always off (color value=0) and the letters can be set by the color selector
const long OMG[256] PROGMEM = {
0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,0,
1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,1,
1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,0,
0,0,0,0,0,1,0,0,0,1,0,0,0,0,0,0,
0,0,0,0,0,1,0,1,0,1,0,1,0,0,0,0,
0,0,0,0,1,0,0,1,0,0,1,0,0,0,0,0,
0,0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,
0,0,0,0,1,0,0,0,0,0,1,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,1,1,1,0,0,
0,0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,1,0,0,1,1,0,
0,1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,1,1,1,0,0
};

const long LOL[256] PROGMEM = {
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,
1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,1,1,1,0,0,0,0,0,1,
1,1,1,1,0,1,0,0,0,1,0,0,0,0,0,0,
0,0,0,0,0,0,1,0,0,0,1,0,0,0,0,0,
0,0,0,0,0,1,0,0,0,1,0,0,0,0,0,0,
0,0,0,0,0,0,1,0,0,0,1,0,0,0,0,0,
0,0,0,0,0,1,0,0,0,1,0,1,0,0,0,0,
0,0,0,0,1,0,0,1,1,1,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,
0,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,
0,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
};

const long Heart[256] PROGMEM = {
1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,
1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,
1,1,0xFF0000,0xFF0000,0xFF0000,1,1,1,1,1,0xFF0000,0xFF0000,0xFF0000,1,1,1,
1,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,1,1,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,1,
0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,
0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,
0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,
1,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,1,
1,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,1,
1,1,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,1,1,
1,1,1,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,1,1,1,
1,1,1,1,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,1,1,1,1,
1,1,1,1,1,0xFF0000,0xFF0000,0xFF0000,0xFF0000,0xFF0000,1,1,1,1,1,1,
1,1,1,1,1,1,0xFF0000,0xFF0000,0xFF0000,1,1,1,1,1,1,1,
1,1,1,1,1,1,1,0xFF0000,1,1,1,1,1,1,1,1,
1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1};

const long Peace[256] PROGMEM = {
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 
0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 
0, 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 
0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 
0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 
0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 
0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 
0, 1, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 0, 1, 0, 0, 
0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 
0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 1, 0, 1, 0, 0, 0, 
0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 
0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 
0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
};

const long CandleImage[256] PROGMEM = {
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0xFFFF00,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0xFFFF00,0,0,0,0,0,0,0,
0,0,0,0,0,0,0xFFFF00,0xFF0000,0xFFFF00,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0xFFFF00,0xFF0000,0xFFFF00,0,0,0,0,0,0,
0,0,0,0,0,0,0xFFFF00,0xFF4500,0xFFFF00,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0xFFFF00,0,0,0,0,0,0,0,
0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,
0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,
0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,
0,0,0x00008B,0,0,0,1,1,1,0,0,0,0,0,0x00008B,0,
0,0,0x00008B,0,0,0,0,1,1,1,0,0,0x00008B,0,0,0,
0,0,0,0,0x00008B,0x00008B,0x00008B,0x00008B,0x00008B,0x00008B,0x00008B,0x00008B,0x00008B,0,0,0
};

const long Poop[256] PROGMEM = {
1,1,1,1,1,1,1,1,1,0x231709,1,1,1,1,1,1,
1,1,1,1,1,0x231709,0x231709,0x231709,0x231709,1,1,1,1,1,1,1,
1,1,1,1,1,1,1,0x231709,0x231709,0x231709,0x231709,0x231709,1,1,1,1,
1,1,1,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,1,1,1,1,
1,1,1,1,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,1,1,1,
1,1,0x231709,0x231709,0xAA4A44,0xAA4A44,0x231709,0x231709,0x231709,0xAA4A44,0xAA4A44,0x231709,0x231709,0x231709,1,1,
1,1,0x231709,0x231709,0x231709,0xAA4A44,0xAA4A44,0x231709,0x231709,0x231709,0xAA4A44,0xAA4A44,0x231709,0x231709,1,1,
1,1,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,1,1,
1,1,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,1,1,
1,0x231709,0x231709,0xAA4A44,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0xAA4A44,0x231709,0x231709,0x231709,1,
1,0x231709,0x231709,0x231709,0x231709,0xAA4A44,0x231709,0x231709,0x231709,0x231709,0x231709,0xAA4A44,0x231709,0x231709,0x231709,1,
0x231709,0x231709,0x231709,0x231709,0x231709,0xAA4A44,0xAA4A44,0xAA4A44,0xAA4A44,0xAA4A44,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,
0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,
0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,
1,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,0x231709,1,
1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1
};

const long BRB[256] PROGMEM = {
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
1,1,1,0,0,0,1,1,1,0,0,0,1,1,1,0,
1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1,
1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1,
1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1,
1,1,1,0,0,0,1,1,1,0,0,0,1,1,1,0,
1,0,0,1,0,0,0,1,0,1,0,0,1,0,0,1,
1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1,
1,0,0,1,0,0,1,0,0,1,0,0,1,0,0,1,
1,1,1,0,0,0,1,0,0,1,0,0,1,1,1,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
};

void setup() { 
FastLED.addLeds<NEOPIXEL,DATA_PIN>(leds, NUM_LEDS);  // Init of the Fastled library
FastLED.setBrightness(15);
pinMode(ImageSwitch_1Pin, INPUT_PULLUP); //initialize all of the pins for the 8-way selector switch as pullup so you don't have use a resistor
pinMode(ImageSwitch_2Pin, INPUT_PULLUP);
pinMode(ImageSwitch_3Pin, INPUT_PULLUP);
pinMode(ImageSwitch_4Pin, INPUT_PULLUP);
pinMode(ImageSwitch_5Pin, INPUT_PULLUP);
pinMode(ImageSwitch_6Pin, INPUT_PULLUP);
pinMode(ImageSwitch_7Pin, INPUT_PULLUP);
pinMode(ImageSwitch_8Pin, INPUT_PULLUP);
pinMode(SendButton, INPUT_PULLUP);

Serial.begin(9600);                   // initialize serial
Serial.println("LoRa Duplex with callback");

  // override the default CS, reset, and IRQ pins (optional)
  LoRa.setPins(csPin, resetPin, irqPin);// set CS, reset, IRQ pin

  if (!LoRa.begin(915E6)) {             // initialize ratio at 915 MHz
    Serial.println("LoRa init failed. Check your connections.");
    while (true);                       // if failed, do nothing
  }

  LoRa.onReceive(onReceive);
  LoRa.receive();
  Serial.println("LoRa init succeeded.");

}

void loop() { 
ColorPotVal = analogRead(ColorPoTPin); //checks the color on the potentiometer
ColorSpot = map((ColorPotVal), 0, 990, 0, 9);

if (digitalRead(ImageSwitch_1Pin)==LOW) 
{
  PoopDisplay(ColorSpot); //if the switch is set to the Poop position, then calls the PoopDisplay function, which draws the Poop emoji. You might have to change these around based on how you wired up your switch to the pins
} 

else if (digitalRead(ImageSwitch_2Pin)==LOW)
{
  HeartDisplay(ColorSpot);
}
else if (digitalRead(ImageSwitch_3Pin)==LOW)
{
  BRBDisplay(ColorSpot);
}
else if (digitalRead(ImageSwitch_4Pin)==LOW)
{
  OMGDisplay(ColorSpot);
}
else if (digitalRead(ImageSwitch_5Pin)==LOW)
{
  SmileyDisplay(ColorSpot);
}

else if (digitalRead(ImageSwitch_6Pin)==LOW)
{
  CandleDisplay(ColorSpot);
}
  else if (digitalRead(ImageSwitch_7Pin)==LOW)
{
  FrownyDisplay(ColorSpot);
}
else if (digitalRead(ImageSwitch_8Pin)==LOW)
{
  LOLDisplay(ColorSpot);
}


delay(10);
//now let's check to see if we are going to send or receive an image

 if (digitalRead(SendButton)== LOW) //if you push the Send button, send the image to other lamp
 {
String message = message + ImageSpot + ColorSpot; //concatenate the Image and Color values into a single message to send
sendMessage(message);
Serial.println("send message ");
Serial.println(message);
message=""; //let's clear out the message for next time
delay(1000);
 }
else
  {
  // parse for a packet, and call onReceive with the result:
  onReceive(LoRa.parsePacket());
  Serial.println("Listening for Message");
  }

} 

//image functions that draw on the matrix
void LOLDisplay(int ColorSpot)
{
FastLED.clear();
ImageSpot = 1; //set the image value that we will send to the other lamp
for(int i = 0; i < NUM_LEDS; i++) {
  
  if (pgm_read_dword(&(LOL[i])) == 1) {  //if the value in the array is 1, then use the color from the color selector
    leds[i] = ColorVal[ColorSpot];
  }
  else{
    leds[i] = (pgm_read_dword(&(LOL[i])));  // If the value is not 1, then use the color value from the array 
  }
}
FastLED.show();

}

void FrownyDisplay(int ColorSpot)
{
FastLED.clear();
ImageSpot = 2;
for(int i = 0; i < NUM_LEDS; i++) {
  
  if (pgm_read_dword(&(Frowny[i])) == 1) {
    leds[i] = ColorVal[ColorSpot];
  }
  else{
    leds[i] = (pgm_read_dword(&(Frowny[i])));  // Read array from Flash
  }
}
FastLED.show();

}

void BRBDisplay(int ColorSpot)
{
FastLED.clear();
ImageSpot = 3;

for(int i = 0; i < NUM_LEDS; i++) {
  
  if (pgm_read_dword(&(BRB[i])) == 1) {
    leds[i] = ColorVal[ColorSpot];
  }
  else{
    leds[i] = (pgm_read_dword(&(BRB[i])));  // Read array from Flash
  }
}
FastLED.show();

}

void PoopDisplay(int ColorSpot)
{
FastLED.clear();
ImageSpot = 4;

for(int i = 0; i < NUM_LEDS; i++) {
  
  if (pgm_read_dword(&(Poop[i])) == 1) {
    leds[i] = ColorVal[ColorSpot];
  }
  else{
    leds[i] = (pgm_read_dword(&(Poop[i])));  // Read array from Flash
  }
}
FastLED.show();

}

void HeartDisplay(int ColorSpot)
{
FastLED.clear();
ImageSpot = 5;

for(int i = 0; i < NUM_LEDS; i++) {
  
  if (pgm_read_dword(&(Heart[i])) == 1) {
    leds[i] = ColorVal[ColorSpot];
  }
  else{
    leds[i] = (pgm_read_dword(&(Heart[i])));  // Read array from Flash
  }
}
FastLED.show();

}

void SmileyDisplay(int ColorSpot)
{
FastLED.clear();
ImageSpot = 6;

for(int i = 0; i < NUM_LEDS; i++) {
  
  if (pgm_read_dword(&(Smiley[i])) == 1) {
    leds[i] = ColorVal[ColorSpot];
  }
  else{
    leds[i] = (pgm_read_dword(&(Smiley[i])));  // Read array from Flash
  }
}
FastLED.show();
/* I added some animation so uncomment out this portion if you want it to wink
delay(500);

leds[68] = 0xFFFF00;
leds[69] = 0xFFFF00;
FastLED.show();
delay(100);
leds[90] = 0xFFFF00;
leds[91] = 0xFFFF00;
FastLED.show();
delay(500);
*/
}

void OMGDisplay(int ColorSpot)
{
FastLED.clear();
ImageSpot = 7;

for(int i = 0; i < NUM_LEDS; i++) {
  
  if (pgm_read_dword(&(OMG[i])) == 1) {
    leds[i] = ColorVal[ColorSpot];
  }
  else{
    leds[i] = (pgm_read_dword(&(OMG[i])));  // Read array from Flash
  }
}
FastLED.show();

}

void CandleDisplay(int ColorSpot)
{
FastLED.clear();
ImageSpot = 8;

for(int i = 0; i < NUM_LEDS; i++) {
  
  if (pgm_read_dword(&(CandleImage[i])) == 1) {
    leds[i] = ColorVal[ColorSpot];
  }
  else{
    leds[i] = (pgm_read_dword(&(CandleImage[i])));  // Read array from Flash
  }
}
FastLED.show();

delay(300); //this animates the candle so it looks like it flickers. you can comment it out if you do not like that effect
leds[54] = 0x000000;
leds[56] = 0x000000;
leds[55] = 0xFFFF00;
leds[40] = 0x000000;
leds[24] = 0x000000;
FastLED.show();
delay(100);
leds[54] = 0xFFFF00;
leds[72] = 0xFFFF00;
leds[55] = 0x000000;
leds[71] = 0x000000;
leds[73] = 0x000000;
FastLED.show();
delay(150);
leds[54] = 0x000000;
leds[72] = 0xFF0000;
leds[55] = 0xFFFF00;
leds[71] = 0xFFFF00;
leds[73] = 0xFFFF00;
leds[86] = 0x000000;
FastLED.show();
delay(100);
}

void PeaceDisplay(int ColorSpot) //bonus image that isn't used
{
FastLED.clear();
ImageSpot = 9;

for(int i = 0; i < NUM_LEDS; i++) {
  
  if (pgm_read_dword(&(Peace[i])) == 1) {
    leds[i] = ColorVal[ColorSpot];
  }
  else{
    leds[i] = (pgm_read_dword(&(Peace[i])));  // Read array from Flash
  }
}
FastLED.show();

}
 
//LoRa functions that came from the LoRa Duplex example code. A lot of it isn't used but I left it in
void sendMessage(String outgoing) {
  LoRa.beginPacket();                   // start packet
  LoRa.write(destination);              // add destination address
  LoRa.write(localAddress);             // add sender address
  LoRa.write(msgCount);                 // add message ID
  LoRa.write(outgoing.length());        // add payload length
  LoRa.print(outgoing);                 // add payload
  LoRa.endPacket();                     // finish packet and send it
  msgCount++;                           // increment message ID
  Serial.println("sending message");
  Serial.println(outgoing);
}


void onReceive(int packetSize) {
  if (packetSize == 0) return;          // if there's no packet, return
  // read packet header bytes:
  int recipient = LoRa.read();          // recipient address
  byte sender = LoRa.read();            // sender address
  byte incomingMsgId = LoRa.read();     // incoming msg ID
  byte incomingLength = LoRa.read();    // incoming msg length
  String incoming = "";
  String imageStr = "";              //used to hold the incoming image position number
  String colorStr = "";              //used to hold the incoming color position number
  while (LoRa.available()) {
   incoming += (char)LoRa.read();;
  }

  if (incomingLength != incoming.length()) {   // check length for error
   Serial.println("error: message length does not match length");
   return;                             // skip rest of function
  }

  // if the recipient isn't this device or broadcast,
  if ((recipient != localAddress) && (recipient != 0xFF)) {
   Serial.println("This message is not for me.");
   return;                             // skip rest of function
  }

 
  // if message is for this device, or broadcast, print details:
  Serial.println("Received from: 0x" + String(sender, HEX));
  Serial.println("Sent to: 0x" + String(recipient, HEX));
  Serial.println("Message ID: " + String(incomingMsgId));
  Serial.println("Message length: " + String(incomingLength));
  Serial.println("Message: " + incoming);
  Serial.println("RSSI: " + String(LoRa.packetRssi()));
  Serial.println("Snr: " + String(LoRa.packetSnr()));
  Serial.println();

imageStr = incoming.charAt(0);  //pull the first character from the incoming message string
ImageSpot = imageStr.toInt();  //this converts the character into an integer so we set the numeric position of the proper image

colorStr = incoming.substring(1);  //since we are only using 0-9 colors, we could use colorStr = incoming.chartAt[1], but the substring function allows use to pull more than one character from the colorStr starting at position 1 (the second character in the string). This way, if we have 10, 100, or more colors, substring will return all of the digits
ColorSpot = colorStr.toInt(); //this converts the character into an integer so we set the numeric position of the proper image
Serial.println(ImageSpot);
Serial.println(ColorSpot);

while (digitalRead(SendButton)== HIGH)  //will stay on sender’s image until you send your own back
{
  
  switch (ImageSpot) {
    case 1:
    LOLDisplay(ColorSpot);
    break;
    case 2:
    FrownyDisplay(ColorSpot);
    break;
    case 3:
    BRBDisplay(ColorSpot);
    break;
    case 4:
    PoopDisplay(ColorSpot);
    break; 
    case 5:
    HeartDisplay(ColorSpot);
    break;
    case 6:
    SmileyDisplay(ColorSpot);
    break;
    case 7:
    OMGDisplay(ColorSpot);
    break;
    case 8:
    CandleDisplay(ColorSpot);
    break;   
  }
} 

}
