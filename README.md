# Project name

This is a modified GNSS-SDR software for AAE6102 (Satellite Navigation).
Author: Baoshan Song.


## Manual

1. git clone https://github.com/222233tommy/AAE6102-assignment1.git;
2. open the project folder in MATLAB;
3. install 'Signal Processing' Toolbox in MATLAB;
4. edit the configuration file 'initSettings.m' to fit your data;
5. run 'init.m' in MATLAB, and the expected result is as follows.

![Expected result](https://github.com/222233tommy/AAE6102-assignment1/blob/main/data/1911741612638_.pic_hd.jpg) 

## Key modules
1. Task 1: Acquisition is done in 'acquisition.m';
  Signal acquisition for GPS is robust in both open-sky and urban environments. 
   
2. Task 2: Tracking is done in 'tracking.m';
   Compared to acquisition, tracking is challenging in urban environments and the multiple correlators can be seriously affected by the multipath effects. The ACF result of one typical epoch polluted by multipath effects is illustrated in Figure 1. In that figure, the peak of the correlators is not shape and compressed due to receiving multiple unhealthy signal at the same time.
   
3. Task 3: Navigation data is decoded in 'include/NAVdecoding.m';
    For the typical urban environment, the IF data is noisy and challenging for Navigation data decoding. Luckily, the Code Division Multiple Access (CDMA) technique helps to lower down the noise and finally we can still get clean ephemerides from the IF data. 
   
4. Task 4: Position is estimated in 'Common/leastSquarePos.m' and Velocity is estimated in 'Common/leastSqaureVel.m';
   Due to multipath/NLOS effect, position/velocity estimation is poor, especially in the east direction (position error in E up to 400 m and velocity error in X up to 30 m/s). The reason can be referred to poor abnormal satellite geometric disrtibution in the skyplot (only 4 satellites in the east part).
   
5. Task 5: Kalman filter-based positioning is done in 'Common/ekf.m', which employs constant velocity model in the prediction phase and Pseudorange/Doppler measurements in measuring update phase.
   If we employ an Extendedn Kalman Filter (EKF) to fuse the measurements, we make both position and velocity error plotting smoother (position error in E smaller than 300 m and velocity error in X smaller than 25 m/s). This is because the noises of both the constant velocity and Doppler measurements are smaller so that the EKF benefits from them. Moreover, the filter parameters align well with real situation, i.e. static mode.
