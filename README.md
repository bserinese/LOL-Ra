# LOL-Ra LED Lamp Project
An LED lamp that displays emojis on a 16x16 LED matrix and sends the image to a partner lamp using the Adafruit LoRa Feather
Welcome to the LOL-Ra Lamp Project by Bryan Serinese
This project uses the Adafruit LoRa Feather to display an image on a 16x16 RGB matrix.  
The 8-way rotary selector allows you to pick 1 of 8 images to display and a potentiometer allows you to select the color.
A send button engages the onboard LoRa radio to transmit the image and color to the partner lamp, which then displays the image 
The partner lamp will continue to display the sent image until it sends its own image back.
You need to add the Adafruit boards to the Board Manager and use Adafruit Feather 32u4
I based the LoRa transmission code off of the LoRa example code:  
LoRa Duplex communication with callback by Tom Igoe created 28 April 2017 
I've also updated the SVG files to laser cut the enclosure for the lamp. There are two versions, one that fits a 16x16 LED matrix and another for an 8x8 LED matrix. 
If you opt for the 8x8 LED matrix, you will have to modify the images to display on an 8x8
