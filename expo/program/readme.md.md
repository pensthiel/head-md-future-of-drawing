Original python code:

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

# Tweaking variables
zoom = 0.75
offset_tweak_left = 185 
offset_tweak_top = -10

# Initialize GPIO settings
GPIO.setmode(GPIO.BCM)
# Define GPIO pins for buttons and LEDs
GPIO.setup(17, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # NEXT FRAME button
GPIO.setup(22, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # PREVIEW button
GPIO.setup(21, GPIO.IN, pull_up_down=GPIO.PUD_UP)  # test/quit button
# Define GPIO pins for LEDs
GPIO.setup(18, GPIO.OUT)  # WHITE LED
GPIO.setup(24, GPIO.OUT)  # RED LED
GPIO.setup(27, GPIO.OUT)  # YELLOW LED
GPIO.setup(23, GPIO.OUT)  # GREEN LED
GPIO.setup(26, GPIO.OUT)  # BIG YELLOW LED
GPIO.setup(16, GPIO.OUT)  # IR LED

# Initialize Pygame
pygame.mixer.pre_init()
pygame.init()

# Get screen dimensions
screen_size = (0, 0)
screen = pygame.display.set_mode(screen_size, pygame.FULLSCREEN)
screen_info = pygame.display.Info()
width = screen_info.current_w
height = screen_info.current_h

# Initialize Picamera2
picam2 = Picamera2()
print("picam init")

# Set the current working directory to the script's location
script_dir = os.path.dirname(os.path.abspath(__file__))
os.chdir(script_dir)

# Load sound files
bell = pygame.mixer.Sound("samples/bell.mp3")
music = pygame.mixer.Sound("samples/music.mp3")
print("bell sound file path:", os.path.abspath("samples/bell.mp3"))
print("music sound file path:", os.path.abspath("samples/music.mp3"))

# Generate a random directory name for frames
rand_int = random.randint(1, 1000)
print(f"Random integer between 1 and 1000: {rand_int}")
os.makedirs(f"frames{rand_int}")
frames_d = (f"frames{rand_int}")

# Initialize key press flags
y_key_pressed = False
w_key_pressed = False
a_key_pressed = False
s_key_pressed = False
d_key_pressed = False

# Initialize frame numbers
frame_number = 0
preview_number = 0

# Configure Picamera2 preview
preview_config = picam2.create_preview_configuration()
picam2.configure(preview_config)
picam2.start()
print("picam2 started")
time.sleep(2)  # Wait for 2 seconds to allow the camera to initialize

# Capture metadata to determine image size
picam2.capture_metadata()
size = picam2.capture_metadata()['ScalerCrop'][2:]
full_res = picam2.camera_properties['PixelArraySize']
size = [int(s * zoom) for s in size]

# Calculate offset
offset_width = (full_res[0] - size[0]) // 2 + offset_tweak_left
offset_height = (full_res[1] - size[1]) // 2 + offset_tweak_top
offset = [offset_width, offset_height]
print(f"offset : {offset}")
picam2.set_controls({"ScalerCrop": offset + size})

# Function to save a frame
def save_frame(directory=frames_d, prefix='frame', file_format='jpg'):
    try:
        global frame_number, frame_to_display
        filename = f"{prefix}_{frame_number}.{file_format}"
        filepath = os.path.join(directory, filename)
        screen.fill((255, 255, 255))
        pygame.display.flip()
        time.sleep(0.1)
        picam2.capture_metadata()
        screen.fill((255, 255, 255))
        pygame.display.flip()
        time.sleep(0.1)
        picam2.capture_file(filepath)
        frame_to_display = filepath
        frame_number += 1
        led_signal()
    except Exception as error:
        print(f"Failed to take and save frame: {error}")

# Function to debounce button presses
def debounce(button_pin):
    time.sleep(0.05)
    return GPIO.input(button_pin)

# Functions to control LEDs
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

# Function to flash LEDs
def led_signal():
    LEDS_off()
    time.sleep(0.05)
    LEDS_on()

# Hide the mouse cursor
pygame.mouse.set_visible(False)
screen.fill((200, 150, 250))
LEDS_on()

try:
    while True:
        pygame.display.flip()

        # Check button presses
        if not debounce(17):
            next_button_pressed = True
        if not debounce(21):
            break
        if not debounce(22):
            preview_button_pressed = True

        # Handle next button press
        if next_button_pressed:
            pygame.mixer.Sound.play(bell)
            save_frame()
            if os.path.exists(frame_to_display):
                try:
                    image = pygame.image.load(frame_to_display)
                    scaled_image = pygame.transform.scale(image, (width, height))
                    screen.blit(scaled_image, (0, 0))
                    pygame.display.flip()
                except Exception as load_error:
                    print(f"Failed to load image: {load_error}")
            led_signal()
            next_button_pressed = False

        # Handle preview button press
        if preview_button_pressed:
            pygame.mixer.Sound.play(music)
            filepath2 = os.path.join(frames_d, f"frame_{preview_number}.jpg")
            preview_number += 1
            if os.path.exists(filepath2):
                try:
                    image = pygame.image.load(filepath2)
                    scaled_image = pygame.transform.scale(image, (width, height))
                    screen.blit(scaled_image, (0, 0))
                    pygame.display.flip()
                    time.sleep(0.1)
                except Exception as file_error:
                    print("Error occurred while loading the image.")
            else:
                print("can't preview: Directory empty")
                preview_button_pressed = False
            if preview_number == frame_number or preview_number == 50:
                preview_number = 0
                pygame.mixer.stop()
                preview_button_pressed = False

        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT or (event.type == pygame.KEYDOWN and event.key == pygame.K_q):
                break
            if event.type == pygame.KEYDOWN and event.key == pygame.K_y:
                if not y_key_pressed:
                    next_button_pressed = True
                    y_key_pressed = True
                if event.type == pygame.KEYUP and event.key == pygame.K_y:
                    y_key_pressed = False
            if event.type == pygame.KEYDOWN and event.key == pygame.K_w:
                if not w_key_pressed:
                    offset_tweak_top += 2
                    w_key_pressed = True
                if event.type == pygame.KEYUP and event.key == pygame.K_w:
                    w_key_pressed = False
            if event.type == pygame.KEYDOWN and event.key == pygame.K_a:
                if not a_key_pressed:
                    offset_tweak_left += 2
                    a_key_pressed = True
                if event.type == pygame.KEYUP and event.key == pygame.K_a:
                    a_key_pressed = False
            if event.type == pygame.KEYDOWN and event.key == pygame.K_d:
                if not d_key_pressed:
                    offset_tweak_left -= 2
                    d_key_pressed = True
                if event.type == pygame.KEYUP and event.key == pygame.K_d:
                    d_key_pressed = False
            if event.type == pygame.KEYDOWN and event.key == pygame.K_s:
                if not s_key_pressed:
                    offset_tweak_top -= 2
                    s_key_pressed = True
                if event.type == pygame.KEYUP and event.key == pygame.K_s:
                    s_key_pressed = False

except KeyboardInterrupt:
    pass

finally:
    # Cleanup and exit
    pygame.mixer.quit()
    GPIO.cleanup()
    picam2.stop()
    pygame.quit()
    quit()

````



to control usb:

````
import uhubctl

hubs = uhubctl.discover_hubs()

for hub in hubs:
    print(f"Found hub: {hub}")

    for port in hub.ports:
        print(f"   Found port: {port}")
        
        # You can use the optional argument `cached_results=False` for each of 
        # these 3 methods in order to invalidate the internal cache,
        # which is used for performance reasons
        print(f"      Description: {port.description()}")
        print(f"      Vendor ID: {port.vendor_id()}")
        print(f"      Product ID: {port.product_id()}")
````

[https://github.com/nbuchwitz/python3-uhubctl](https://github.com/nbuchwitz/python3-uhubctl)

````
from uhubctl import Hub, Port

hub = Hub("1-1")
port = hub.add_port(1)

print("Switch port 1-1.1 off")
port.status = False

print("Switch port 1-1.1 on")
port.status = True

print("Get port 1-1.1 status")
print(port.status)
````

add "move the house !" frame

