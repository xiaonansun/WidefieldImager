# WidefieldImager
This repo contains Matlab-based acquisition software and a pre-processing pipeline for widefield imaging. 
The software was used for the publication 'Single-trial neural dynamics are dominated by richly varied movements' by Musall and Kaufman et al.. 

Pre-processed imaging data from the study can be found here: http://repository.cshl.edu/id/eprint/38599/

See also https://github.com/jcouto/wfield for python code with extended functionality.

Some of the pre-processing analysis was inspired by code from Kenneth Harris and Matteo Carandini's lab: https://github.com/cortex-lab/widefield

# Overview
The 'WidefieldImager' folder contains the widefield acquisition software. 'LEDswitcher' contains code for a Teensy microcrontoller (https://www.pjrc.com/store/teensy32.html) to control different excitation LEDs in the widefield setup. 
The 'CAD' folder contains 3-D models for different parts that are useful for head-fixed imaging, such as headbars, a clamp and a light-shielding cone.
The 'Analysis' folder contains a Matlab pipeline to perfornm linear dimensionality reduction (using localized singular value decomposition) and hemodynamic correction on raw imaging data. 

There are no specific hardware requirements. To capture imaging data, it is beneficial to use a larger (>=1TB) solid state drive and sufficient RAM (>=16GB). Since data is streamed to memory and written to disc between trials, this is particularly important when imaging for longer durations or high frame rates.
The pre-processing pipeline also benefits from sufficient RAM (>= 16GB). The optional GPU-acceleration requires a CUDA compatible GPU from Nvidia, ideally with at least 6GB of VRAM.


# Widefield acquisition software
The software is customized to work with a PCO.edge 5.5 camera. You can modify WidefieldImager.m to work with different camera systems through Matlabs ImageAcquisition toolbox.
Before starting the code, make sure to install all required PCO drivers, the Matlab adaptor package and the CamWare software. In Camware, set the Exposure trigger to 'All lines' and close again before starting Matlab.

Clone or download the repository to your imaging PC and add it to your Matlab path. Type 'WidefieldImager' to run the acqusition software. Click “New animal” and enter animal ID
when prompted. Make sure other settings, such as trial duration and file path are correct. Click “start preview”, center the animal under the objective and switch on the blue light.
Position the light shielding cone, if needed.

Click “Take Snapshot” to store an image of the preparation before starting the recording. To start imaging, change the excitation light to “Mixed light” and click the “Wait for trigger” button. The software will no wait for a stimulus trigger, from a behavioral or stimulation system.
Start the stimulation/behavioral paradigm. To start a trial, a “trial start” trigger needs to be sent to the NI-DAQ and imaging data will be recorded when a stimulus trigger is received. 
The data includes frames, according to the baseline and a post-stimulus period. If no stimulus trigger is received within the ‘Wait’ period, the trial is stopped and a new “trial start” trigger needs to be provided. An optional “trial end” trigger can be provided to stop a trial immediately. 
Check the MATLAB command window to ensure that the correct number of frames are saved in each trial. To stop the recording, click the “Locked” and then the “Wait for Trigger” button.
This will stop the recording after the current trial is completed or no stimulus trigger has been received within the 'Wait' duration.


# Widefield pre-processing software
Clone or download the repository to your analysis PC and add it to your Matlab path. You can either analysis your own imaging data or download a demo recording here.
Navigate to the folder that contains your imaging data in a folder named 'DemoRec', then type 'edit Tutorial_dimReduction.m'.
Tutorial_dimReduction is a demo scrip that contains different variables that are relevant for pre-processing. All variables are described in the script and should be self-explanatory. Write an issue report if there are any problems and I'll try to assist.

The pipeline has multiple steps: blockSVD separates the imaging frames into smaller blocks and performs linear dimensionality using randomized SVD. This steps returns block-wise data 'bV' and 'bU'.
A second SVD is used to isolate common temporal dimensions across all blocks, representing the main temporal components for the whole data set. These components 'nV' are then used to compute the corresponding whole-frame spatial components 'U'.
Lastly, 'SvdHemoCorrect' performs hemo-dynamic correction on the low-dimensional data by regressing out fluoresence with violet illumination from blue illumination frames.

The resulting dataset 'Vc' represents the corrected imaging dataset. To restore individual frames simply transpose U and Vc (second dimension of Vc are frames). 
For example

```rawData = U * Vc(:, 1:10)```

will restore the first 10 frames in the current dataset. 
The 'index frameCnt' contains the number of frames per trial and 'stimTime' indicates at which trial a stimulus was presented'. Using these variables you can reconstruct imaging data from different trials or responses to stimulus events of interest.