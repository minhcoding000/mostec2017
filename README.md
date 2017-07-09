import RPi.GPIO as GPIO
import pygame
import time

GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

#set up pins
inputs = [12,16,20,21]
#for each pin set it as an input!
for x in inputs:
    GPIO.setup(x,GPIO.IN,pull_up_down=GPIO.PUD_UP)

#song list
songs = ['genghis_khan','shake_off','di de','wash_side','queen_janelle','yes']
song_loc = '/home/pi/mostec17/audio/'
pygame.mixer.init()

current_song = 0 #used to count through songs!
play_pause_state = False #True is playing, #False is not playing

#create list...initialize with whatever values:
old_switch_states= [True,True,True,True]

#initialize switch state values!
for x in range(len(inputs)):
    #print(inputs[x])
    old_switch_states[x] = GPIO.input(inputs[x])

#preload a song, play it, immediately pause it!
pygame.mixer.music.load(song_loc+songs[current_song]+'.mp3')
pygame.mixer.music.play()
pygame.mixer.music.pause()

def unpause():
    global play_pause_state #global allows to change variable inside function
    play_pause_state = True
    print("Unpause")
    pygame.mixer.music.unpause()

def pause():
    global play_pause_state
    play_pause_state = False
    print("Pause")
    pygame.mixer.music.pause()
def next_song():
    global current_song
    current_song = current_song + 1
    print(current_song)
    print(len(songs))
    if current_song == len(songs):
        print("Reached Maximum")
        current_song = 0

    print("Next song")   
    pygame.mixer.music.load(song_loc+songs[current_song]+'.mp3')
    pygame.mixer.music.play()

def volume(inc_dec):
    print("changing volume")
    percent = 0.1
    curVol = pygame.mixer.music.get_volume()
    print(curVol)
    if inc_dec and curVol != 1:
        #Increase
        pygame.mixer.music.set_volume(curVol+percent)
    elif curVol != 0:
        #Decrease
        curVol = curVol - percent
        pygame.mixer.music.set_volume(curVol-percent)
    pass

def readInputs(new_readings, old_readings):
    a = len(old_readings)
    if old_readings[a-1] == True and new_readings[a-1] == False:
        return "next_song"
    elif old_readings[a-2] == True and new_readings[a-2] == False:
        return "pause_unpause"
    elif old_readings[a-3] == True and new_readings[a-3] == False:
        return "volup"
    elif old_readings[a-4] == True and new_readings[a-4] == False:
        return "voldown"

try:
    while True:
        new_switch_states = []
        for x in inputs:
            new_switch_states.append(GPIO.input(x))
        output = readInputs(new_switch_states,old_switch_states)
        old_switch_states=new_switch_states
        if output != None:
            print(output)
        if output == 'pause_unpause' and not play_pause_state:
            unpause()
        elif output == 'pause_unpause' and play_pause_state:
            pause()
        elif output == 'next_song':
            next_song()
        elif output == 'volup':
            volume(True)
        elif output == 'voldown':
            volume(False)
        time.sleep(0.1)
except KeyboardInterrupt:
    pygame.mixer.music.stop()
    GPIO.cleanup()
