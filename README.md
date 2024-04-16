***TRACEBOT***
This repository contains Python code for controlling a 3-axis GRBL stage together with a either Piezo pumps (Bartel's MP-6) or miniature peristaltic pumps (Jobst CPP1) and their synchronization with a microscope, currently implemented for a Nikon instrument.
See instructions below for installation and usage. 
Please have a look at our manuscript (https://doi.org/10.1101/2021.04.12.439407) for details on the setup and application.

**Setup robot PC and instrument PC**

Connect laptop PC power, 3-axis stage power, pump power (for Bartel’s pumps), and USB from stage and pump to PC.
Ch340 drivers usually need to be installed to control the stage, Bartel’s pump drivers as well, CPP pumps do not need separate drivers.

Install environment for running tracebot and microscopy automation software on both robot PC and microscope PC:

Mount (or access) network drive that both PCs can access.

Here we activate a virtual environment (e.g., created with `virtualenv`), and install `tracebot`'s dependencies:
```console
cd /path/to/shared/folder && source /path/to/your/venv/bin/activate
git clone https://github.com/gerlichlab/tracebot.git
cd tracebot && pip install -r requirements.txt
```

**Robot setup**
 
Duplicate configs/robot_config_template.yaml then rename the new file to make a custom config file. Change parameters in file as needed, most parameters should be documented in template config file. COM ports can be verified in Device Manager on the laptop. 

Start the gui, with virtual environment into which you've installed Tracebot's dependencies already activated, and from the tracebot code folder:
```console
python ./mw_pump_gui.py
```

In the GUI, select the custom config file, and press “Initialize robot”. If all is correct, stage and pump should connect.

Test the pump by setting a time (e.g. 10 s) and pressing Pump cycle.

Before moving the stage the first time, press “Zero stage”.

If 3-axis stage is not already above well A1, manually screw the z-axis screw upward until ~1-2 mm from top. Can do this by controlling the z, y and x with the small remote control (will disconnect from PC when you do this), or manually driving the stage (but be careful not to crash!).

Press “Zero stage”.

Manually screw (or carefully drive using coordinates maintaining z=0 and adjusting x and y) until inlet is just above well A1 in the 96-well plate (upper right corner on stage base when facing the stage).

Press “Zero stage” again. 

The calibration and custom positions can be tested using “Move to position”.

Once the stage, pump and sequential setup have been verified, pressing “Start robot” will run defined sequence. 

Unchecking “Restart?” will let the robot continue at the previously selected well from an earlier run rather than at the first well indicated in the configuration file. 

“Stop robot” will allow halting the sequence at any point after the completion of the current command. Note however that upon restarting, the sequence will restart at the first command of the sequence, although the correct well plate position will be continued if unchecking “Restart?” as above.

**Setting up microscope PC (Nikon)**

Start the automation software (remember this has to be in the same folder as the robot PC is running from), from the Tracebot code folder and with Tracebot's virtual environment activated:
```
python ./auto_image_nikon_jobs.py
```

In the NIS-elements software, open/edit the macro found in /path/to/shared/folder/tracebot/nikon/pythonnissync_events.mac in this repository and run it to set the two functions (pnsSetImageAvailable(), pnsWaitForImageProcessed()). These two functions will create two win32 events, set them and monitor them for changes. These two events are also read/written by auto_image_nikon_jobs.py. In the workflow (typically a JOBS pipeline), run these two functions as appropriate, typically pnsWaitForImageProcessed() at the start of a loop sequence, and pnsSetImageAvailable() at the end.

Define the rest of the job with appropriate microscope settings.

**Running an automated sequential experiment**

With the robot and microscope set up, the acquisition can be started by running the job in NIS-elements (the first step will be to wait for input from the robot), then starting the robot from the robot GUI. Once the 'image: 1' command is reached in the sequence, the image acquisition loop will be triggered, and once complete the robot will be triggered and proceed with the defined sequence.
