# samplerboxrpi5
install notes for https://github.com/hansehv/SamplerBox on a raspberry pi 5

hello all, this is a place for me to organize how i installed hansehv's most recent version of SamplerBox on a Raspberry Pi 5 (i think it was compatible with up till RPI3 or 4).

long story short, there is a couple of dependencies which are a bit finnicky to install, and i modified some of the code to get rid of depreciated stuff.

you can start with regular raspberry pi 5 image, i used the instructions here to get an RT-PREEMPT version (lower latency).  it's quite a bit of extra compiling to make the image https://github.com/remusmp/rpi-rt-kernel

i suggest you read this file in Raw, when you read it on githubs normal website it eliminates a lot of carriage returns and smushes things onto one line


step by step of the actual samplerbox install:

**making virtual environment**
mkdir 3rdtry
cd 3rdtry
python3 -m venv argh
source argh/bin/activate

**cloning samplerbox**
git clone https://github.com/hansehv/SamplerBox
cd SamplerBox

**dependencies**
pip3 install setuptools
pip3 install Cython
pip3 install setuptools
pip3 install numpy
python3 setup.py build_ext --inplace

**rtmidi2 problems, i think maybe i just ended up replacing this file with the one i have in this repo but i'm keeping this part in case it actually was useful **

git clone https://github.com/hansehv/rtmidi2
cd rtmidi2
python3 setup.py install (****((this does a compileerror))
**need to compile the interface.so file**
g++ -Wall -Wextra -O -ansi -pedantic -fpermissive -shared interface.cpp -o interface.so -fPIC
**this has the added -fPIC which was an error thrown when used without**
git clone https://github.com/hansehv/python-midi
cd python-midi
python3 setup.py install
pip install git+https://github.com/hansehv/python-midi

***error  modules/smfplayer.py", line 52, in <module> AttributeError: module 'midi.sequencer' has no attribute 'SequencerWrite' ** i think i just commented that line out.. maybe i don<t have smfplayer working i dunno **

pip3 install sounddevice
pip3 install pyalsaaudio
pip install rpi.gpio

** FileNotFoundError: [Errno 2] No such file or directory: 'config/controls.csv' ** **need to rename Controls.csv to controls.csv **

sudo apt install liblgpio-dev
pip3 install rpi-lgpio


***issues with sounds.py (chunk problems) i saved all the fixes of sounds.py, i'm a bad person, i used chatGPT to help me fix it. strangely it seems to still use Chunk, but works for me.  feel free to troubleshoot further, basically chunk is a depreciated library, it<s been replaced with waveread. long story short use my sounds.py and should work**


** replaced a ton of stuff in rtmidi2.pyx i think i also used ChatGPT to "fix" it, see attached file .. this is just to get stuff running a better programmer than me needs to take a look at any changes.  i'm using samplerbox with GPIO inputs and not midi so i'm not even sure if midi works properly, i would assume so but thought i would put full disclosure here**


------------
that should be it.  test it before you do this part, the last part is to make it auto-boot the code when you boot the RPI
you can run it within the venv without activating the venv, in fact i think it's the only way to get it to run properly on boot, so try out this way of running it (no need to activate venv):
/home/pi/3rdtry/argh/bin/python3 /home/pi/3rdtry/SamplerBox/samplerbox.py
(this uses the python3 program in the venv directly)



**for auto boot** 

Step 1: Create a systemd Service File

    Open a new service file using sudo nano:
    bash

sudo nano /etc/systemd/system/my_script.service

Paste the following configuration, replacing the placeholder paths and username with your specific details [1]:
ini

[Unit]
Description=My Script Service
After=network.target

[Service]
User=pi 
WorkingDirectory=/home/pi/my_project 
ExecStart=ExecStart=sudo chrt --fifo 99 /home/pi/3rdtry/argh/bin/python3 /home/pi/3rdtry/SamplerBox/samplerbox.py
R
Restart=always

[Install]
WantedBy=multi-user.target

    User=: Replace pi with your Raspberry Pi username.
    WorkingDirectory=: The absolute path to your project directory.
    ExecStart=: The absolute path to your venv's Python executable followed by the absolute path to your script [1].
    After=network.target: Ensures the network is ready before attempting to start the script, which is important for IoT or network-dependent applications [1]. 

Step 3: Enable and Start the Service 

    Save and exit the nano editor (Ctrl+O, Enter, Ctrl+X).
    Reload systemd to recognize the new service:
    bash

sudo systemctl daemon-reload

Enable the service to start automatically at boot:
bash

sudo systemctl enable my_script.service

Start the service immediately without rebooting:
bash

sudo systemctl start my_script.service

 

Step 4: Verify the Service Status
You can check if the script is running correctly at any time using this command: 
bash

sudo systemctl status my_script.service


if you want to stop it use systemctl stop my_script.service

** the above uses chrt --fifo 99 for running the program, since i'm using RT-PREEMPT, it will set real-time scheduling policy (essential if you want low latency with RT-PREEMPT).  if you're just using regular RPI5, just run it with  /home/pi/3rdtry/argh/bin/python3 /home/pi/3rdtry/SamplerBox/samplerbox.py .. you might need a . before the above line i never remember that stuff **

hope that helps, feel free to add any notes of things that need to be changed / explained better

