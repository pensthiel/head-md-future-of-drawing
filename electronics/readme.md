# Electronic

Shortcuts:

1. [2023-12-06](#2023-12-06) Setting up the Rasberry PI

<br/><br/>

###### 2023-12-06

## Setting up the Rasberry PI

I plugged the PI to power and a screen but nothing appeared so I watched [this video](https://www.youtube.com/watch?v=2RHuDKq7ONQ) to flash the sd card. IT WORKED:

![pi](photos/IMG_1219.JPG)


<br/>


###### 2023-12-06

## Soldering ex.

![solder](photos/IMG_1321.JPG)  
![solder2](photos/IMG_1323.JPG)


<br/>


###### 2023-12-21

## controlling the projector

TRY to control it with HDMI with a Python library.

plan B :  
OCTO-ISOLATORs connected to buttons to control the projector buttons with the pi, you activate it as if u were turning on a led.

Don't forget we'll add LEDs to the house !




<br/>


###### 2024-01-09

## To do



- Projector control solution

I noticed there's an IR sensor and found the remote, I could read the remote's signals with the flipper zero so that I can send those signals from the rasberry with a IR led :-)



#### componants list



- <b><span style="color:rgba(244,63,94,1)">Pi</span></b>

**power switch**

GPIO pins 39 and 40. -> [link](https://www.makeuseof.com/tag/add-power-button-raspberry-pi/#:~:text=Mount%2520a%2520Raspberry%2520Pi%2520Off%2520Switch%2520on%2520the%2520GPIO&text=If%2520you%2520can't%2520get,GPIO%2520pins%252039%2520and%252040.)



**cherry button next frame**

GPIO 17 & GND -> [link](http://razzpisampler.oreilly.com/ch07.html)



**cherry button preview movie**

GPIO 18 & GND



**IR led**

[link](https://projects.raspberrypi.org/en/projects/infrared-bird-box/5) , [link](https://asimuzzaman.com/posts/send-infrared-ir-remote-signal-with-python) (directly to the 5 volt supply of the Raspberry Pi with a 220 ohm resistor inline)



**house led x4?**

[link](https://www.maskaravivek.com/post/controlling-multiple-leds-using-raspberry-pis-gpio-ports/) GPIO 27, 22, 23, 24

![](photos/GPIO-Pinout-Diagram-2.png)



###### 2024-01-10

## Fritzing

![](fritzing/breadboard.png)![](fritzing/schematic.png)

Needs to be verified by pierre


<br>


###### 2024-01-13

## Final schematic

Here's what we got :

![](photos/final2.png)![](photos/final1.png)
![](fritzing/Screenshot 2024-01-15 at 20.17.51.png>)


<br>


###### 2024-01-16

## Object

![photos](photos/img_1551.jpg)
![photo2](photos/img_1548.jpg)