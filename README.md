# Project name

    This is a repository for AAE6102 assignment 1 (Satellite Navigation). 

    Author: Baoshan Song.

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

The task aims to identify visible satellites and estimate coarse values of carrier Doppler frequency and code phase for each satellite signal. To achieve this efficiently, we implemented a parallel code phase search method based on Fast Fourier Transform (FFT). After conditioning the input signal to remove DC bias, we established a two-dimensional search space covering a Doppler range of ±5 kHz (with 500 Hz steps) and the full code phase range (1023 chips).&#x20;

For each PRN, we generated local C/A code replicas and transformed them into the frequency domain using FFT. The acquisition algorithm then performed circular cross-correlation by multiplying the frequency-domain representations of the incoming signal and local replicas, followed by an inverse FFT to obtain correlation results. This approach allows simultaneous evaluation of all code phases for each Doppler bin, significantly improving computational efficiency. Satellite detection was based on comparing the highest correlation peak with the second highest, using a threshold derived from the desired false alarm probability. For successfully detected satellites, refined Doppler frequency estimates were obtained through fine frequency estimation using longer integration periods, which also addressed differences in sampling and intermediate frequencies between open-sky and urban datasets.

The results of acquisition are shown as below.

1\) Open-sky test

2\) Urban test

# Task 2: Tracking of SDR

    Tracking is done in 'tracking.m';

In this task, to refine the coarse estimates obtained during acquisition, the tracking process employs feedback loops that continuously follow satellite signals. In this work, we focused on evaluating the effects of urban interference on the correlation function shape by implementing a multi-correlator architecture. Our system integrates a Phase Lock Loop (PLL) with a Costas discriminator for carrier tracking and a non-coherent Delay Lock Loop (DLL) for code tracking.

To enable detailed analysis of the correlation function, we extended the traditional Early-Prompt-Late (EPL) configuration to a nine-correlator setup with 0.1-chip spacing, covering a range of ±0.4 chips around the prompt position. This enhanced correlator array provides higher resolution in visualizing the correlation function, allowing for more effective detection of multipath distortions. The DLL utilizes a non-coherent early-minus-late power discriminator to robustly track code phase under challenging signal conditions.

These multiple correlation points enable detailed visualization of the auto-correlation function (ACF), providing insight into signal quality and multipath presence. The ACF results of acquisition are shown as below.

1\) Open-sky test

In the Open-sky test, the autocorrelation function (ACF) displays symmetric and well-defined peaks across all measurement epochs, indicating clean line-of-sight signal reception with minimal distortion. The observed symmetry confirms the absence of significant multipath effects in the open-sky environment.

2\) Urban test

# Task 3: Navigation data decoding

    Navigation data is decoded in 'include/NAVdecoding.m';

In this task, we provide the results in extracting the navigation message by enabling bit synchronization and frame structure decoding. After appropriate filtering and bit synchronization, the in-phase (I-prompt) values reveal the 50 Hz navigation data embedded in the signal. The navigation data decoding process involves several key steps: performing bit synchronization to identify data bit boundaries, detecting the preamble for frame synchronization, identifying subframes, and decoding their contents. Parity checks are applied to ensure data integrity. Once these steps are completed, essential information such as ephemeris parameters, time of week (TOW), and other broadcast data can be successfully extracted from the signal.

1\) Open-sky test





2\) Urban test



In summary, although the IF data is noisy and challenging for Navigation data decoding. Luckily, the Code Division Multiple Access (CDMA) technique helps to lower down the noise and finally we can still get clean ephemerides from the IF data.

# Task 4 & 5: Weighted Least Squares (WLS)-based and extended Kalman-filter (EKF)-based navigation

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

Note that the weighting model from PANDA software is applied in this study, where \alpha = 3 mm.

```math
W=\frac {1}{(\alpha+sin^2(EL))^2}, EL>30\degree; or \frac{1}{\alpha^2}, EL<30\degree.

```



    Kalman filter-based positioning is done in 'Common/ekf.m', which employs constant velocity model in the prediction phase and Pseudorange/Doppler measurements in measuring update phase. Here we aim to estimate the position and velocity together in an extended Kalman filter (EKF). Based on EKF, we can get a smoother solution.

```math
X=[p_x,p_y,p_z, v_x,v_y,v_z, c\cdot d t,c\cdot d\dot t]^T
```

We evaluate the SPP and SPV using both open-sky and urban datasets. Their results are shown in the following figures. Note that we just found that the signs of Doppler shift measurements are opposite in open-sky and urban datasets, thus we converted them to the same before velocity estimation.&#x20; The results of WLS-based and EKF-based state estimation are shown as below:

1\) Open-sky dataset:

WLS-based positioning
![wls_pos](https://github.com/222233tommy/AAE6102-assignment1/blob/main/assignment1/spp_pos.png)


EKF-based positioning
<img src="https://raw.githubusercontent.com/222233tommy/AAE6102-assignment1/main/assignment1/ekf_pos.png" style="zoom: 25%;" />

WLS-based vs. EKF-based velocity error
![ekf_spv_vel](https://github.com/222233tommy/AAE6102-assignment1/blob/main/assignment1/velocity_error_ekf_spv.png)



For the Open-sky test, position error in E smaller than 300 m and velocity error in X smaller than 25 m/s). We consider that the positioning results benefit from two constraints: the constant velocity systematic model constraint and the low-noise Doppler measurements constraint.

2\) Urban dataset:

According to the results of the Urban test, positioning errors are limited and there are less zig-zag in the positioning error figure (position error in E smaller than 300 m and velocity error in X smaller than 25 m/s). This is because the noises of both the constant velocity and Doppler measurements are smaller so that the EKF benefits from them. Moreover, the filter parameters align well with real situation, i.e. static mode.

In summary, both experiments have explored the potential of using velocity to smooth position estimation. It has proved that if the systematic models and measuring models are consistent with the truth, the estimation of the state in a filter could be more precise.
