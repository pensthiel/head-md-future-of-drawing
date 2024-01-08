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
```

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
```
It seems that on the py it wasn't displaying the right picture because it was taking too much time to save on the drive, but I added a delay to give it some time.

### next
I still haven't scripted the video playback and at the end I'll have to reassign the buttons because right now it triggers with the keyboard.