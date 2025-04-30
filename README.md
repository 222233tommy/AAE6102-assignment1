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

| PRN      | 16          | 22          | 26          | 27          | 31          |
|----------|-------------|-------------|-------------|-------------|-------------|
| C_ic     | -1.01E-07   | -1.01E-07   | -2.05E-08   | 1.08E-07    | -1.14E-07   |
| omega_0  | -1.674261429| 1.272735322 | -1.812930701| -0.71747466 | -2.787272903|
| C_is     | 1.36E-07    | -9.31E-08   | 8.94E-08    | 1.15E-07    | -5.03E-08   |
| i_0      | 0.971603403 | 0.936454583 | 0.939912327 | 0.974727542 | 0.95588255  |
| C_rc     | 237.6875    | 266.34375   | 234.1875    | 230.34375   | 240.15625   |
| omega    | 0.679609497 | -0.887886686| 0.295685419 | 0.630881665 | 0.311626182 |
| omegaDot | -8.01E-09   | -8.67E-09   | -8.31E-09   | -8.02E-09   | -7.99E-09   |
| IODE_sf3 | 9           | 22          | 113         | 30          | 83          |
| iDot     | -4.89E-10   | -3.04E-11   | -4.18E-10   | -7.14E-13   | 3.21E-11    |
| idValid  | [2,0,3]     | [2,0,3]     | [2,0,3]     | [2,0,3]     | [2,0,3]     |
| weekNumber| 1155       | 1155        | 1155        | 1155        | 1155        |
| accuracy | 0           | 0           | 0           | 0           | 0           |
| health   | 0           | 0           | 0           | 0           | 0           |
| T_GD     | -1.02E-08   | -1.77E-08   | 6.98E-09    | 1.86E-09    | -1.30E-08   |
| IODC     | 234         | 218         | 15          | 4           | 228         |
| t_oc     | 396000      | 396000      | 396000      | 396000      | 396000      |
| a_f2     | 0           | 0           | 0           | 0           | 0           |
| a_f1     | -6.37E-12   | 9.21E-12    | 3.98E-12    | -5.00E-12   | -1.93E-12   |
| a_f0     | -0.000406925| -0.000489472| 0.00014479  | -0.000206121| -0.0001449  |
| IODE_sf2 | 9           | 22          | 113         | 30          | 83          |
| C_rs     | 23.34375    | -99.8125    | 21.25       | 70.4375     | 30.71875    |
| deltan   | 4.25E-09    | 5.28E-09    | 5.05E-09    | 4.03E-09    | 4.81E-09    |
| M_0      | 0.718116855 | -1.260965589| 1.735570934 | -0.173022281| 2.82452322  |
| C_uc     | 1.39E-06    | -5.16E-06   | 1.15E-06    | 3.73E-06    | 1.46E-06    |
| e        | 0.012296279 | 0.006713538 | 0.006253509 | 0.009574107 | 0.010271554 |
| C_us     | 7.69E-06    | 5.17E-06    | 7.04E-06    | 8.24E-06    | 7.23E-06    |
| sqrtA    | 5153.771322 | 5153.712273 | 5153.636459 | 5153.652021 | 5153.622389 |
| t_oe     | 396000      | 396000      | 396000      | 396000      | 396000      |
| TOW      | 390102      | 390102      | 390102      | 390102      | 390102      |




2\) Urban test

| PRN        | 1            | 3            | 11           | 18          |
| ---------- | ------------ | ------------ | ------------ | ----------- |
| C_ic       | -7.45E-08    | 1.12E-08     | -3.17E-07    | -2.53E-07   |
| omega_0    | -3.106035801 | -2.064178438 | 2.725770376  | 3.121821254 |
| C_is       | 1.60E-07     | 5.22E-08     | -1.32E-07    | 3.54E-08    |
| i_0        | 0.976127704  | 0.962858746  | 0.909806736  | 0.9546426   |
| C_rc       | 287.46875    | 160.3125     | 324.40625    | 280.15625   |
| omega      | 0.711497599  | 0.594974558  | 1.891492962  | 1.393015876 |
| omegaDot   | -8.17E-09    | -7.83E-09    | -9.30E-09    | -8.61E-09   |
| IODE_sf3   | 72           | 72           | 83           | 56          |
| iDot       | -1.81E-10    | 4.81E-10     | 1.29E-11     | -1.62E-10   |
| idValid    | [2,0,3]      | [2,0,3]      | [2,0,3]      | [2,0,3]     |
| weekNumber | 1032         | 1032         | 1032         | 1032        |
| accuracy   | 0            | 0            | 0            | 0           |
| health     | 0            | 0            | 0            | 0           |
| T_GD       | 5.59E-09     | 1.86E-09     | -1.26E-08    | -5.59E-09   |
| IODC       | 12           | 4            | 229          | 244         |
| t_oc       | 453600       | 453600       | 453600       | 453600      |
| a_f2       | 0            | 0            | 0            | 0           |
| a_f1       | -9.44E-12    | -1.14E-12    | 8.53E-12     | 3.18E-12    |
| a_f0       | -3.49E-05    | 0.000186326  | -0.000590093 | 5.99E-05    |
| IODE_sf2   | 72           | 72           | 83           | 56          |
| C_rs       | -120.71875   | -62.09375    | -67.125      | -113.875    |
| deltan     | 4.19E-09     | 4.45E-09     | 5.89E-09     | 4.72E-09    |
| M_0        | 0.517930888  | -0.430397464 | -0.198905418 | 0.259840989 |
| C_uc       | -6.33E-06    | -3.09E-06    | -3.60E-06    | -6.11E-06   |
| e          | 0.008923085  | 0.00222623   | 0.016643139  | 0.015419818 |
| C_us       | 5.30E-06     | 1.16E-05     | 1.51E-06     | 5.11E-06    |
| sqrtA      | 5153.655643  | 5153.777802  | 5153.706596  | 5153.699318 |
| t_oe       | 453600       | 453600       | 453600       | 453600      |
| TOW        | 449352       | 449352       | 449352       | 449352      |


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
