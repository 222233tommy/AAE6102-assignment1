# Project name

This is a modified GNSS-SDR software for AAE6102 (Satellite Navigation).


## Manual

1. git clone https://github.com/222233tommy/AAE6102-assignment1.git;
2. open the project folder in MATLAB;
3. install 'Signal Processing' Toolbox in MATLAB;
4. edit the configuration file 'initSettings.m' to fit your data;
5. run 'init.m' in MATLAB, and the expected result is as follows.

![Expected result](https://github.com/222233tommy/AAE6102-assignment1/blob/main/data/1911741612638_.pic_hd.jpg) 

## Key modules
1. Task 1: Acquisition is done in 'acquisition.m';
2. Task 2: Tracking is done in 'tracking.m';
3. Task 3: Navigation data is decoded in 'include/NAVdecoding.m';
4. Task 4: Position is estimated in 'Common/leastSquarePos.m' and Velocity is estimated in 'Common/leastSqaureVel.m';
5. Task 5: Kalman filter-based positioning is done in 'Common/ekf.m', which employs constant velocity model in the prediction phase and Pseudorange/Doppler measurements in measuring update phase.
