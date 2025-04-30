# Project name



    This is a modified GNSS-SDR software for AAE6102 (Satellite Navigation). Author: Baoshan Song.
    NetID: 23093571R

This study is mainly about the analysis of GNSS signal processing using a Software-Defined Receiver (SDR) approach. The objective is to process and analyze real Intermediate Frequency (IF) datasets collected in two different environments: an Open-sky and a typical Urban environment. The implementation and analysis focus on five tasks: acquisition, tracking, navigation data decoding, position/velocity estimation using Weighted Least Squares (WLS), and state estimation using an extended Kalman Filter (EKF).



If you want to run the codes used in this study, please follows these steps:

1.  git clone <https://github.com/222233tommy/AAE6102-assignment1.git>;
2.  open the project folder in MATLAB;
3.  install 'Signal Processing' Toolbox in MATLAB;
4.  edit the configuration file 'initSettings.m' to fit your data;
5.  run 'init.m' in MATLAB, and the expected result is as follows.



# &#x20;Task 1: Acquisition of SDR

     Acquisition is done in 'acquisition.m';



Signal acquisition for GPS is robust in both open-sky and urban environments.

# Task 2: Tracking of SDR

    Tracking is done in 'tracking.m';

For the typical urban environment, the IF data is noisy and challenging for Navigation data decoding. Luckily, the Code Division Multiple Access (CDMA) technique helps to lower down the noise and finally we can still get clean ephemerides from the IF data.



# Task 3: Navigation data decoding

    Navigation data is decoded in 'include/NAVdecoding.m';





# Task 4: Weighted Least Squares (WLS)-based navigation

    Position is estimated in 'Common/leastSquarePos.m' and Velocity is estimated in 'Common/leastSqaureVel.m';

In the following experiments we define all the parameters in the Earth-centered, Earth-fixed (ECEF) frame. The principal state estimation is based on the WLS formula here:

```math
\Delta X = (H^TWH)^{-1}H^TWZ
```

Note that standard point positioning (SPP) using WLS estimator, the H, W and Z should be calculated iteratively.&#x20;



```math
X=[p_x,p_y,p_z,c\cdot d t]^T 
```



Compared to SPP, the standard point velocity (SPV) using WLS does not require iteration, as its raw measuring function is linear. Therefore, we set the state as X:

```math
X=[v_x,v_y,v_z,c\cdot d\dot t]^T
```

Both SPP and SPV using GPS require at least 4 measurements to guarantee the degree of freedom (DoF) in the estimation.



Note that the weighting model from PANDA software is applied in this study.

```math
W=\frac {1}{(\alpha+sin^2(EL))^2}, EL>30\degree; \frac{1}{\alpha^2}, EL<30\degree.

```

We evaluate the SPP and SPV using both open-sky and urban datasets. Their results are shown in the following figures. Note that we just found that the signs of Doppler shift measurements are opposite in open-sky and urban datasets, thus we converted them to the same before velocity estimation.&#x20;



# Task 5: Extended Kalman filter (EKF)-based navigation

    Kalman filter-based positioning is done in 'Common/ekf.m', which employs constant velocity model in the prediction phase and Pseudorange/Doppler measurements in measuring update phase.

In this task, we aim to estimate the position and velocity together in an extended Kalman filter (EKF). Based on EKF, we can get a smoother solution.

```math
X=[p_x,p_y,p_z, v_x,v_y,v_z, c\cdot d t,c\cdot d\dot t]^T
```

The results of EKF-based state estimation are shown as below:



1\) Open-sky dataset:





For the Open-sky test, position error in E smaller than 300 m and velocity error in X smaller than 25 m/s). We consider that the positioning results benefit from two constraints: the constant velocity systematic model constraint and the low-noise Doppler measurements constraint.

2\) Urban dataset:





According to the results of the Urban test, positioning errors are limited and there are less zig-zag in the positioning error figure (position error in E smaller than 300 m and velocity error in X smaller than 25 m/s). This is because the noises of both the constant velocity and Doppler measurements are smaller so that the EKF benefits from them. Moreover, the filter parameters align well with real situation, i.e. static mode.



In summary, both experiments have explored the potential of using velocity to smooth position estimation. It has proved that if the systematic models and measuring models are consistent with the truth, the estimation of the state in a filter could be more precise.



