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


<br>


###### 2024-01-08

## Working script 1

#### (to display the pictures taken)

````

import cv2
import pygame
from pygame.locals import *
import os
import random

#how much time we give the pi to save the image
SaveDelay = 2000 # 2 seconds in milliseconds

# Use Pygame's clock to handle the timing
clock = pygame.time.Clock()
start_ticks = pygame.time.get_ticks()  # Starter tick

# Set the current working directory to the script's location
script_dir = os.path.dirname(os.path.abspath(__file__))
os.chdir(script_dir)

# Initialize Pygame
pygame.init()

# Set display size
screen_size = (0, 0)  # Set to (0, 0) for full screen
screen = pygame.display.set_mode(screen_size, pygame.FULLSCREEN)

# Get the current display info
screen_info = pygame.display.Info()
width = screen_info.current_w
height = screen_info.current_h

# Initialize the camera
cap = cv2.VideoCapture(0)  # Use 0 for the default camera

# Fetch the frame width and height
frame_width = cap.get(cv2.CAP_PROP_FRAME_WIDTH)
frame_height = cap.get(cv2.CAP_PROP_FRAME_HEIGHT)

# Calculate the ratio
ratio = frame_width / frame_height
new_width = int(height * ratio)

# Frame count initialization
frame_number = 0
# Generate a random integer from 1 to 10
rand_int = random.randint(1, 1000)
print(f"Random integer between 1 and 1000: {rand_int}")

# Create 'frames' directory if it doesn't exist
os.makedirs(f"frames{rand_int}")
frames_d = (f"frames{rand_int}")

y_key_pressed = False

frame_to_display = None

# Function to save the frame
def save_frame(image, directory=frames_d, prefix='frame', file_format='jpg'):
    global frame_number, frame_to_display  # Declare both as global
    filename = f"{prefix}_{frame_number}.{file_format}"
    filepath = os.path.join(directory, filename)
    cv2.imwrite(filepath, image)
    print(f"{filepath} Saved")
    frame_to_display = filepath
    frame_number += 1  # Increment the frame number

def play_vid(frame):
    frame = cv2.resize(frame, (new_width, height))
    img = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    img = pygame.surfarray.make_surface(img.swapaxes(0, 1))
    screen.blit(img, (0, 0))
    pygame.display.flip()
    print("playing video")

try:
    while True:
        seconds = (pygame.time.get_ticks() - start_ticks) / 1000
        image_loaded = False
        ret, frame = cap.read()

        # Handle events
        for event in pygame.event.get():
            if event.type == pygame.QUIT or (event.type == pygame.KEYDOWN and event.key == pygame.K_q):
                print("q to quit")
                raise StopIteration  # Break out of the loop

            # Check for 'y' key press to save the frame
            if event.type == pygame.KEYDOWN and event.key == pygame.K_y:
                print("key event detected")
                if not y_key_pressed:  # Check if the 'Y' key was not already pressed
                    screen.fill((255, 255, 255))
                    pygame.display.flip()  # Ensure the screen updates before capturing the frame
                    save_frame(frame)  # Save the frame with an auto-incremented number


                    # Delay before loading and displaying the image
                    save_delay_ticks = SaveDelay  
                    start_save_delay_ticks = pygame.time.get_ticks()

                    while pygame.time.get_ticks() - start_save_delay_ticks < save_delay_ticks:
                        pygame.event.pump()
                        clock.tick(60)

                    try:
                        image = pygame.image.load(frame_to_display)
                        print(frame_to_display + " loaded")
                        image = pygame.transform.scale(image, (new_width, height))
                        screen.blit(image, (0, 0))
                        pygame.display.flip()
                        image_loaded = True
                        print(frame_to_display + " displayed")

                    except Exception as e:
                        print(f"Failed to load image: {e}")

                    y_key_pressed = True  # Set the variable to True after the action


        if event.type == pygame.KEYUP and event.key == pygame.K_y:
            y_key_pressed = False  # Reset the variable when the 'Y' key is released

except StopIteration:
    pass  # Exit the loop when 'q' is pressed or the window is closed

finally:
    # Release resources
    pygame.quit()
    cap.release()
````

It seems that on the py it wasn't displaying the right picture because it was taking too much time to save on the drive, but I added a delay to give it some time.

### next

I still haven't scripted the video playback and at the end I'll have to reassign the buttons because right now it triggers with the keyboard.



to install to pi:

````
sudo pip3 install gpiozero
````

[this](https://www.makeuseof.com/tag/add-power-button-raspberry-pi/#:~:text=Mount%252520a%252520Raspberry%252520Pi%252520Off%252520Switch%252520on%252520the%252520GPIO&text=If%252520you%252520can't%252520get,GPIO%252520pins%25252039%252520and%25252040.)

````
$ sudo apt-get install python-rpi.gpio python3-rpi.gpi



````

````

sudo nano /boot/config.txt
````

There, find these lines:

````
#dtoverlay=gpio-ir,gpio_pin=17
#dtoverlay=gpio-ir-tx,gpio_pin=18
````

And uncomment only those lines to enable IR transmission:

````
dtoverlay=gpio-ir,gpio_pin=17
dtoverlay=gpio-ir-tx,gpio_pin=18
````

Now restart the Raspberry Pi by running `sudo reboot`.

````
sudo apt update
sudo apt install lirc -y
````

sudo nano /etc/lirc/lirc_options.conf

````
driver          = default
device          = /dev/lirc0
````

`sudo reboot` and after rebooting, check whether the LIRC daemon is running or not.

````
sudo /etc/init.d/lircd status
````

you will see you will see `**active (running)**`



pip install lirc

<br>

###### 2024-01-08

## GPIO mapping and Preview program



So this code works about the wiring is unstable on the buttons, I wanted to be able to unplug replug but that won't work

````
import cv2
import pygame
from pygame.locals import *
from gpiozero import Button
import RPi.GPIO as GPIO # Import Raspberry Pi GPIO library
import os
import random



GPIO.setmode(GPIO.BCM)     # set up BCM GPIO numbering  

GPIO.setup(17, GPIO.IN, pull_up_down=GPIO.PUD_UP)   # NEXT FRAME
GPIO.setup(22, GPIO.IN, pull_up_down=GPIO.PUD_UP)   # PREVIEW

GPIO.setup(21, GPIO.IN, pull_up_down=GPIO.PUD_UP)   # shutdown?


GPIO.setup(18, GPIO.OUT)   # output (LED)  WHITE
GPIO.setup(24, GPIO.OUT)    # RED
GPIO.setup(27, GPIO.OUT)    # YELLOW
GPIO.setup(23, GPIO.OUT)    # GREEN

GPIO.setup(16, GPIO.OUT) # IR 


# Initialize Pygame
pygame.init()

# LED ON yELLOW
GPIO.output(27,GPIO.HIGH) 




#how much time we give the pi to save the image
SaveDelay = 2000 # 2 seconds in milliseconds
VidFrameRate = 100

# Use Pygame's clock to handle the timing
clock = pygame.time.Clock()
start_ticks = pygame.time.get_ticks()  # Starter tick

# Set the current working directory to the script's location
script_dir = os.path.dirname(os.path.abspath(__file__))
os.chdir(script_dir)



# Set display size
screen_size = (0, 0)  # Set to (0, 0) for full screen
screen = pygame.display.set_mode(screen_size, pygame.FULLSCREEN)

# Get the current display info
screen_info = pygame.display.Info()
width = screen_info.current_w
height = screen_info.current_h

# Initialize the camera
cap = cv2.VideoCapture(0)  # Use 0 for the default camera

# Fetch the frame width and height
frame_width = cap.get(cv2.CAP_PROP_FRAME_WIDTH)
frame_height = cap.get(cv2.CAP_PROP_FRAME_HEIGHT)

# Calculate the ratio
ratio = frame_width / frame_height
new_width = int(height * ratio)

# Frame count initialization
frame_number = 0
preview_number = 0
# Generate a random integer from 1 to 10
rand_int = random.randint(1, 1000)
print(f"Random integer between 1 and 1000: {rand_int}")

# Create 'frames' directory if it doesn't exist
os.makedirs(f"frames{rand_int}")
frames_d = (f"frames{rand_int}")

y_key_pressed = False

frame_to_display = None
next_button_pressed = False
preview_button_pressed = False
filepath2 = None


# Function to save the frame
def save_frame(image, directory=frames_d, prefix='frame', file_format='jpg'):
    global frame_number, frame_to_display  # Declare both as global
    filename = f"{prefix}_{frame_number}.{file_format}"
    filepath = os.path.join(directory, filename)
    cv2.imwrite(filepath, image)
    print(f"{filepath} Saved")
    frame_to_display = filepath
    frame_number += 1  # Increment the frame number


#def play_vid(frame):
#   frame = cv2.resize(frame, (new_width, height))
#   img = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
#   img = pygame.surfarray.make_surface(img.swapaxes(0, 1))
#   screen.blit(img, (0, 0))
#   pygame.display.flip()
#   print("playing video")



try:

    while True:
        seconds = (pygame.time.get_ticks() - start_ticks) / 1000
        image_loaded = False
        ret, frame = cap.read()


        # TEST
        if not GPIO.input(21): 
            print("black button is LOW (pressed), playing the next frame event as test")
            next_button_pressed = True

        # NEXT BUTTON
        if not GPIO.input(17): # if port 17 == 0  
            print("next frame button is LOW (pressed)")
            next_button_pressed = True
        # PREVIEW 
        if not GPIO.input(22): 
            print("preview button is LOW (pressed)")
            preview_button_pressed = True

        if next_button_pressed:

                screen.fill((255, 255, 255))
                pygame.display.flip()  # Ensure the screen updates before capturing the frame
                save_frame(frame)  # Save the frame with an auto-incremented number

                # Delay before loading and displaying the image
                save_delay_ticks = SaveDelay  
                start_save_delay_ticks = pygame.time.get_ticks()

                while pygame.time.get_ticks() - start_save_delay_ticks < save_delay_ticks:
                    pygame.event.pump()
                    clock.tick(60)

                try:
                    image = pygame.image.load(frame_to_display)
                    print(frame_to_display + " loaded")
                    image = pygame.transform.scale(image, (new_width, height))
                    screen.blit(image, (0, 0))
                    pygame.display.flip()
                    image_loaded = True
                    print(frame_to_display + " displayed")

                except Exception as e:
                    print(f"Failed to load image: {e}")
                
                # Delay before u can press the button again
                save_delay_ticks = SaveDelay  
                start_save_delay_ticks = pygame.time.get_ticks()
                while pygame.time.get_ticks() - start_save_delay_ticks < save_delay_ticks:
                    pygame.event.pump()
                    clock.tick(60)
                next_button_pressed = False  # Set the variable to True after the action
        


        # PREVIEW 
        if preview_button_pressed:

            filepath2 = os.path.join(frames_d, f"frame{preview_number}.jpg")
            image = pygame.image.load(filepath2)
            print(filepath2 + " loaded")
            image = pygame.transform.scale(image, (new_width, height))
            screen.blit(image, (0, 0))
            pygame.display.flip()
            image_loaded = True
            print(frame_to_display + " displayed")
            preview_number += 1
            VidFrameRate_ticks = VidFrameRate 
            start_VidFrameRate_ticks = pygame.time.get_ticks()
            while pygame.time.get_ticks() - start_VidFrameRate_ticks < VidFrameRate_ticks:
                pygame.event.pump()
                clock.tick(60)


            if preview_number > frame_number:
                preview_number = 0
                preview_button_pressed = False






        # Handle events
        for event in pygame.event.get():
            if event.type == pygame.QUIT or (event.type == pygame.KEYDOWN and event.key == pygame.K_q):
                print("q to quit")
                raise StopIteration  # Break out of the loop
        
    

            # Check for 'y' key press to save the frame
            if event.type == pygame.KEYDOWN and event.key == pygame.K_y:
                print("key event detected")
                if not y_key_pressed:  # Check if the 'Y' key was not already pressed
                    screen.fill((255, 255, 255))
                    pygame.display.flip()  # Ensure the screen updates before capturing the frame
                    save_frame(frame)  # Save the frame with an auto-incremented number

                    # Delay before loading and displaying the image
                    save_delay_ticks = SaveDelay  
                    start_save_delay_ticks = pygame.time.get_ticks()

                    while pygame.time.get_ticks() - start_save_delay_ticks < save_delay_ticks:
                        pygame.event.pump()
                        clock.tick(60)

                    try:
                        image = pygame.image.load(frame_to_display)
                        print(frame_to_display + " loaded")
                        image = pygame.transform.scale(image, (new_width, height))
                        screen.blit(image, (0, 0))
                        pygame.display.flip()
                        image_loaded = True
                        print(frame_to_display + " displayed")

                    except Exception as e:
                        print(f"Failed to load image: {e}")

                    y_key_pressed = True  # Set the variable to True after the action
            


            
            



            if event.type == pygame.KEYUP and event.key == pygame.K_y:
                y_key_pressed = False  # Reset the variable when the 'Y' key is released




except StopIteration:
    pass  # Exit the loop when 'q' is pressed or the window is closed


finally:
    # Release resources
    pygame.quit()
    cap.release()
    GPIO.cleanup()

````



###### 2024-01-13

## New cam new programe

Okay so we finally received the componants but when I replaced the usb cam by the PI cam 3 (because the other one couldnt do what i needed) I ran into a big problem : 

Pi cameras work very differantly, thought I needed picamera but it dont work so picamera2 but I couldnt install, saw its not compatible on some OS but then after 3 days fo debugging and trying things i finally got picamera 2 to work, turns out it doesnt work in a virtual enviroment so i switched to installing my pakages system wide and ran my script there.



Then It's a whole other libabry so I had to reado the code from scratch and AIs weren't much help because picamera2 is pretty recent so i had to go in and read the whole [doc](chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/https://datasheets.raspberrypi.com/camera/picamera2-manual.pdf)



after some trials, errors and lots of debugging i got that:

````
#!/usr/bin/python3

# Import necessary libraries
from picamera2 import Picamera2, Preview
import RPi.GPIO as GPIO
import os
import random
import time
import pygame
from pygame.locals import *

GPIO.setmode(GPIO.BCM)
GPIO.setup(17, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # NEXT FRAME
GPIO.setup(22, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # PREVIEW
GPIO.setup(21, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # test/quit button
GPIO.setup(18, GPIO.OUT)  # output (LED)  WHITE
GPIO.setup(24, GPIO.OUT)  # RED
GPIO.setup(27, GPIO.OUT)  # YELLOW
GPIO.setup(23, GPIO.OUT)  # GREEN
GPIO.setup(16, GPIO.OUT)  # IR

# Initialize Pygame
pygame.init()
screen_size = (0, 0)  # Set to (0, 0) for full screen
screen = pygame.display.set_mode(screen_size, pygame.FULLSCREEN) # Set display size
screen_info = pygame.display.Info() # Get the current display info
width = screen_info.current_w
height = screen_info.current_h

# Create a Picamera2 instance
picam2 = Picamera2()
print("picam init")

zoom = 0.95 # copped image /1


# Set the current working directory to the script's location
script_dir = os.path.dirname(os.path.abspath(__file__))
os.chdir(script_dir)

# Generate a random integer from 1 to 1000
rand_int = random.randint(1, 1000)
print(f"Random integer between 1 and 1000: {rand_int}")

# Create 'frames' directory if it doesn't exist
os.makedirs(f"frames{rand_int}")
frames_d = (f"frames{rand_int}")

y_key_pressed = False
frame_to_display = None
next_button_pressed = False
preview_button_pressed = False
filepath = None
filepath2 = None

# Frame count initialization
frame_number = 0
preview_number = 0


preview_config = picam2.create_preview_configuration()
picam2.configure(preview_config)
picam2.start()
print("picam2 started")
# Wait for 2 seconds to allow the camera to initialize
time.sleep(2)

picam2.capture_metadata()
size = picam2.capture_metadata()['ScalerCrop'][2:]
full_res = picam2.camera_properties['PixelArraySize']
picam2.capture_metadata()
size = [int(s * zoom) for s in size]
offset = [(r - s) // 2 for r, s in zip(full_res, size)]
picam2.set_controls({"ScalerCrop": offset + size})


def save_frame(directory=frames_d, prefix='frame', file_format='jpg'):
    try:
        global frame_number, frame_to_display  # Declare both as global
        filename = f"{prefix}_{frame_number}.{file_format}"
        print(filename)
        filepath = os.path.join(directory, filename)
        print(filepath)
        screen.fill((255, 255, 255))
        picam2.capture_metadata()
        picam2.capture_file(filepath)
        frame_to_display = filepath
        frame_number += 1
        led_signal()
    except Exception as error:
        print(f"Failed to take and save frame: {error}")

    

def debounce(button_pin):
    time.sleep(0.05)  # Adjust the sleep time based on your requirements
    return GPIO.input(button_pin)

def LEDS_on():
    GPIO.output(18, GPIO.HIGH)
    GPIO.output(24, GPIO.HIGH)
    GPIO.output(27, GPIO.HIGH)
    GPIO.output(23, GPIO.HIGH)

def LEDS_off():
    GPIO.output(18, GPIO.LOW)
    GPIO.output(24, GPIO.LOW)
    GPIO.output(27, GPIO.LOW)
    GPIO.output(23, GPIO.LOW)

def led_signal():
    LEDS_off()
    time.sleep(0.05)
    LEDS_on()

screen.fill((200, 150, 250))
LEDS_on()
try:
    while True:
        pygame.display.flip()


       
        if not debounce(17):
            print("next button pressed")
            next_button_pressed = True
        
        if not debounce(21):
            print("exit button pressed")
            break

        if not debounce(22):
            print("preview button pressed")
            preview_button_pressed = True
        

        if next_button_pressed:
            print("next frame starts")
            try:
                screen.fill((255, 255, 255))
                save_frame()
            except Exception as next_frame_error:
                print(f"couldn't complete save_frame {next_frame_error}")
            try:
                if os.path.exists(frame_to_display):  # Checking for file existence outside the loop can speed things up significantly
                    try:
                        image = pygame.image.load(frame_to_display)
                        scaled_image = pygame.transform.scale(image, (width, height))
                        screen.blit(scaled_image, (0, 0)) 
                        pygame.display.flip()
                        led_signal()
                    
                    except Exception as load_error:
                        print(f"Failed to load image: {load_error}")
                    
            except Exception as file_error:
                print("Error occurred while loading image.")  # Use error handling to catch and report any issues smoothly.

            next_button_pressed = False

        if preview_button_pressed:
            print("preview starts")
            filepath2 = os.path.join(frames_d, f"frame_{preview_number}.jpg")
            preview_number += 1
            print(filepath2)
            if os.path.exists(filepath2):  # Checking for file existence outside the loop can speed up significantly
                try:
                    if not image is None and not image.get_rect().size == (0, 0):
                        image = pygame.image.load(filepath2)
                        scaled_image = pygame.transform.scale(image, (width, height))
                        print(filepath2 + " loaded")
                        screen.blit(scaled_image, (0, 0))
                        print(filepath2 + " displayed")
                        time.sleep(0.1)
                
                except Exception as file_error:
                    print("Error occurred while loading the image.")   # Use error handling to catch and report any issues smoothly.
          
            else:
                print("can't preview: Directory empty")
                preview_button_pressed = False

            if preview_number > frame_number:
                preview_number = 0
                preview_button_pressed = False

        for event in pygame.event.get():
            if event.type == pygame.QUIT or (event.type == pygame.KEYDOWN and event.key == pygame.K_q):
                led_signal()
                print("q to quit")
                break
            
            if event.type == pygame.KEYDOWN and event.key == pygame.K_y:
                print("y key event detected")
                if not y_key_pressed:
                    print("next frame triggered with y key")
                    next_button_pressed = True
                    y_key_pressed = True  # Set the variable to True after the action
                    
                if event.type == pygame.KEYUP and event.key == pygame.K_y:
                    y_key_pressed = False  # Reset the variable when the 'Y' key is released


finally:
    # Release resources
    GPIO.cleanup()
    picam2.stop()
    pygame.quit()
    quit()


````

<br>

###### 2024-01-16

## FINAL CODE

Added audio and fixed some stuff
````
#!/usr/bin/python3

# Import necessary libraries
from picamera2 import Picamera2, Preview
import RPi.GPIO as GPIO
import os
import random
import time
import pygame
from pygame.locals import *

zoom = 0.75 # copped image /1
offset_tweak_left = 195  # Change this value as needed
offset_tweak_top = -60  # Change this value as needed

GPIO.setmode(GPIO.BCM)
GPIO.setup(17, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # NEXT FRAME
GPIO.setup(22, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # PREVIEW
GPIO.setup(21, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # test/quit button
GPIO.setup(18, GPIO.OUT)  # output (LED)  WHITE
GPIO.setup(24, GPIO.OUT)  # RED
GPIO.setup(27, GPIO.OUT)  # YELLOW
GPIO.setup(23, GPIO.OUT)  # GREEN
GPIO.setup(26, GPIO.OUT)  # big yellow
GPIO.setup(16, GPIO.OUT)  # IR

pygame.mixer.pre_init()
# Initialize Pygame
pygame.init()
screen_size = (0, 0)  # Set to (0, 0) for full screen
screen = pygame.display.set_mode(screen_size, pygame.FULLSCREEN) # Set display size
screen_info = pygame.display.Info() # Get the current display info
width = screen_info.current_w
height = screen_info.current_h

# Create a Picamera2 instance
picam2 = Picamera2()
print("picam init")



# Set the current working directory to the script's location
script_dir = os.path.dirname(os.path.abspath(__file__))
os.chdir(script_dir)

bell = pygame.mixer.Sound("samples/bell.mp3")
music = pygame.mixer.Sound("samples/music.mp3")
#pygame.mixer.Sound.play(bell)
#pygame.mixer.music.stop()
print("bell sound file path:", os.path.abspath("samples/bell.mp3"))
print("music sound file path:", os.path.abspath("samples/music.mp3"))
#pygame.mixer.Sound.play(music)
#pygame.mixer.music.stop()

# Generate a random integer from 1 to 1000
rand_int = random.randint(1, 1000)
print(f"Random integer between 1 and 1000: {rand_int}")

# Create 'frames' directory if it doesn't exist
os.makedirs(f"frames{rand_int}")
frames_d = (f"frames{rand_int}")

y_key_pressed = False
w_key_pressed = False
a_key_pressed = False
s_key_pressed = False
d_key_pressed = False
frame_to_display = None
next_button_pressed = False
preview_button_pressed = False
filepath = None
filepath2 = None

# Frame count initialization
frame_number = 0
preview_number = 0


preview_config = picam2.create_preview_configuration()
picam2.configure(preview_config)
picam2.start()
print("picam2 started")
# Wait for 2 seconds to allow the camera to initialize
time.sleep(2)


picam2.capture_metadata()
size = picam2.capture_metadata()['ScalerCrop'][2:]
full_res = picam2.camera_properties['PixelArraySize']
picam2.capture_metadata()
size = [int(s * zoom) for s in size]
#offset = [(r - s) // 2 for r, s in zip(full_res, size)]
#picam2.set_controls({"ScalerCrop": offset + size})

# Calculate offset based on the initial values
offset_width = (full_res[0] - size[0]) // 2 + offset_tweak_left
offset_height = (full_res[1] - size[1]) // 2 + offset_tweak_top

# Create a list with the individual offset values
offset = [offset_width, offset_height]
print(f"offset : {offset}")
# Set controls with individual offset values
picam2.set_controls({"ScalerCrop": offset + size})








def save_frame(directory=frames_d, prefix='frame', file_format='jpg'):
    try:
        global frame_number, frame_to_display  # Declare both as global
        filename = f"{prefix}_{frame_number}.{file_format}"
        print(filename)
        filepath = os.path.join(directory, filename)
        print(filepath)

        # Fill the screen with a white background
        screen.fill((255, 255, 255))
        pygame.display.flip()  # Update the display
        time.sleep(0.1)
        picam2.capture_metadata()
        screen.fill((255, 255, 255))
        pygame.display.flip()  # Update the display a second time
        time.sleep(0.1)
        picam2.capture_file(filepath)
        frame_to_display = filepath
        frame_number += 1
        led_signal()
    except Exception as error:
        print(f"Failed to take and save frame: {error}")

    

def debounce(button_pin):
    time.sleep(0.05)  # Adjust the sleep time based on your requirements
    return GPIO.input(button_pin)

def LEDS_on():
    GPIO.output(18, GPIO.HIGH)
    GPIO.output(24, GPIO.HIGH)
    GPIO.output(27, GPIO.HIGH)
    GPIO.output(23, GPIO.HIGH)
    GPIO.output(26, GPIO.HIGH)

def LEDS_off():
    GPIO.output(18, GPIO.LOW)
    GPIO.output(24, GPIO.LOW)
    GPIO.output(27, GPIO.LOW)
    GPIO.output(23, GPIO.LOW)
    GPIO.output(26, GPIO.LOW)

def led_signal():
    LEDS_off()
    time.sleep(0.05)
    LEDS_on()


screen.fill((200, 150, 250))
LEDS_on()
try:
    while True:
        pygame.display.flip()


       
        if not debounce(17):
            print("next button pressed")
            next_button_pressed = True
        
        if not debounce(21):
            print("exit button pressed")
            break

        if not debounce(22):
            print("preview button pressed")
            preview_button_pressed = True
        

        if next_button_pressed:
            print("next frame starts")
            try:
                pygame.mixer.Sound.play(bell)
            except Exception as e:
                print("Error playing sound:", e)
            try:
                save_frame()
            except Exception as next_frame_error:
                print(f"couldn't complete save_frame {next_frame_error}")
            try:
                if os.path.exists(frame_to_display):  # Checking for file existence outside the loop can speed things up significantly
                    try:
                        image = pygame.image.load(frame_to_display)
                        scaled_image = pygame.transform.scale(image, (width, height))
                        screen.blit(scaled_image, (0, 0)) 
                        pygame.display.flip()
                        
                    
                    except Exception as load_error:
                        print(f"Failed to load image: {load_error}")
                    
            except Exception as file_error:
                print("Error occurred while loading image.")  # Use error handling to catch and report any issues smoothly.

            led_signal()
            next_button_pressed = False

        if preview_button_pressed:
            print("preview starts")
            try:
                pygame.mixer.Sound.play(music)
            except Exception as e:
                print("Error playing sound:", e)
            filepath2 = os.path.join(frames_d, f"frame_{preview_number}.jpg")
            preview_number += 1
            print(filepath2)
            if os.path.exists(filepath2):  # Checking for file existence outside the loop can speed up significantly
                try:
                    if not image is None and not image.get_rect().size == (0, 0):
                        image = pygame.image.load(filepath2)
                        scaled_image = pygame.transform.scale(image, (width, height))
                        print(filepath2 + " loaded")
                        screen.blit(scaled_image, (0, 0))
                        print(filepath2 + " displayed")
                        time.sleep(0.1)
                
                except Exception as file_error:
                    print("Error occurred while loading the image.")   # Use error handling to catch and report any issues smoothly.
          
            else:
                print("can't preview: Directory empty")
                preview_button_pressed = False

            if preview_number > (frame_number - 1):
                preview_number = 0
                pygame.mixer.sound.stop(music)
                preview_button_pressed = False

            if preview_number > 50:
                preview_number = 0
                pygame.mixer.sound.stop()
                preview_button_pressed = False

        for event in pygame.event.get():
            if event.type == pygame.QUIT or (event.type == pygame.KEYDOWN and event.key == pygame.K_q):
                led_signal()
                print("q to quit")
                break
            
            if event.type == pygame.KEYDOWN and event.key == pygame.K_y:
                print("y key event detected")
                if not y_key_pressed:
                    print("next frame triggered with y key")
                    next_button_pressed = True
                    y_key_pressed = True  # Set the variable to True after the action
                    
                if event.type == pygame.KEYUP and event.key == pygame.K_y:
                    y_key_pressed = False  # Reset the variable when the 'Y' key is released
            
            if event.type == pygame.KEYDOWN and event.key == pygame.K_w:
                if not w_key_pressed:
                    print("up")
                    #offset_tweak_left = 320  # Change this value as needed
                    offset_tweak_top += 2

                    print(f": {offset_tweak_top}")
                    w_key_pressed = True  # Set the variable to True after the actio
                if event.type == pygame.KEYUP and event.key == pygame.K_w:
                    w_key_pressed = False  # Reset the variable when the 'Y' key is released

            if event.type == pygame.KEYDOWN and event.key == pygame.K_a:
                if not a_key_pressed:
                    print("left")
                    offset_tweak_left += 2  # Change this value as needed
       
                    print(f"left: {offset_tweak_left}")
                    a_key_pressed = True  # Set the variable to True after the actio
                if event.type == pygame.KEYUP and event.key == pygame.K_a:
                    a_key_pressed = False  # Reset the variable when the 'Y' key is released

            if event.type == pygame.KEYDOWN and event.key == pygame.K_d:
                if not d_key_pressed:
                    print("right")
                    offset_tweak_left -= 2 # Change this value
    
                    print(f"left: {offset_tweak_left}")
                    d_key_pressed = True  # Set the variable to True after the actio
                if event.type == pygame.KEYUP and event.key == pygame.K_d:
                    d_key_pressed = False  # Reset the variable when the 'Y' key is released

            if event.type == pygame.KEYDOWN and event.key == pygame.K_s:
                if not s_key_pressed:
                    print("down")
                    #offset_tweak_left -= 2 # Change this value
                    offset_tweak_top -= 2

                    print(offset_tweak_top)
                    s_key_pressed = True  # Set the variable to True after the actio
                if event.type == pygame.KEYUP and event.key == pygame.K_s:
                    s_key_pressed = False  # Reset the variable when the 'Y' key is released
            

except KeyboardInterrupt:
    pass  # Handle the Ctrl+C interrupt to gracefully exit the program


finally:
    # Release resources
    pygame.mixer.quit()
    GPIO.cleanup()
    picam2.stop()
    pygame.quit()
    quit()

````