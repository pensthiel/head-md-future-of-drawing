# UI and Programming

 [2023-12-08](#2023-12-08) Research

<br/><br/>

###### 2023-12-08
## Research

Doug recommended I use OpenCV to use the camera as imput instead of physical buttons.
I found [this](https://github.com/amarlearning/Finger-Detection-and-Tracking) but it's in PYTHON, I would like to do everything in python to learn it but teachers will probably recommend using Processing java...

Is Processing Python an option ?????

<br/>

###### 2023-12-13
## Programming day 1

Processing is no good for handling video feeds, I found that out on my first test so I switched to Python. I also realized it has more resources online that are also more accessible.

Hadn't done Python in a while so GPT helped me ease back into it.

1. PI

Had to setup the PI but I had the 32bit version which was no good so I re-flashed it to 64bit

2. github

This was harder than anticipated because git won't let you log in with a password in the terminal anymore. So I had to use SSH which was confusing but Jonas helped me.

The PI is Linux and doesn't have GitHub desktop support so I learned to use the terminal too.

3. Remote Desktop setup

I usually use DWservices which had PI support but it wasn't working at first because of XWayland. Fixed it using raspi-config.

4. Python and OpenCV

I has able to get a simple script with a virtual environment running pretty fast, thanks to the courses I had with Vitas last year.

I tested out an example of using color detection to activate buttons using OpenCV and I got it working but I feel like it wouldn't be reliable enough for use on my projects and it's still better to use real buttons.



There's also not enough space on the screen so if I were putting buttons there they would get activated by mistake.

However, the idea of adding storytelling or instructions on the projected image would still be a good idea and with shape detection like I tested I could detect when someone starts drawing.

5. Base onionskinning program for test day

I tried to make a program, it had to display the video stream and then pictures full screen. I did not succeed in doing that just with openCV, in the end, I used Pygame and some other things..

I had some difficulty coding at first but with that exercise, my skills got a lot better already.


### Things to watch out for

- Need to add zoom lenses or something to the camera so that in can be next to the projector. also will need a lot of fine-tuning.

- the code could be improved, made less resource-intensive, as the PI is having trouble.

- will need to add a blank screen when taking the pic 

- gotta make the program to display the film 

- i wanna add sound too

