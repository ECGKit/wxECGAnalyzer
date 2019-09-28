# wxECGAnalyzer
Detection of abnormal rhythm morphologies is more difficult than normal beat, therefore, we need to collect abnormal rhythm signals in clinical practice for improve QRS-Complex detection algorithm.</br>
This project is for Electrocardiogram(ECG) signal algorithms design and validation, include preprocessing, QRS-Complex detection, embedded system validation, ECG segmentation, label your machine learning dataset, and clinical trial...etc.</br>
</br>
For algorithm performance, in ANSI/AAMI EC38,it is required that the detected QRS shall in the 150ms range of the signed point from annotation by human exper.
</br>
</br>

## Use
### Dependence
- Win10
  - wxWidgets 3.1.2
  - VS2017 - MSVC 10.0.17763-SDK
- Mac High Sierra
  - wxWidgets 3.x
  - g++
### Build
- Win10
  - 1.Open wxECGAnalyzer.sln
  - 2.Rebuild
![alt text](https://github.com/GCY/wxECGAnalyzer/blob/master/res/win10.PNG?raw=true)  
- Mac High Sierra
  - 1.Make
![alt text](https://github.com/GCY/wxECGAnalyzer/blob/master/res/high%20sierra.png?raw=true) 
### Operate Manual
  - Clinical trial
    - 1.Setup your ECG device to  clinical trial.
    - 2.Connect VCP to wxECGAnalyzer. (Tools -> VCP, select cu or COM devices,bouadrate is UART only)
    - 3.Monitor target.
    - 4.Segmentation and save target morphology of the ECG.(you can modify the windows-size, tihs project is 700ms)
    - 5.Select [ECG-Codes](https://github.com/GCY/wxECGAnalyzer/blob/master/src/MAC/define.h) to labeling.
    
[Snapshot.csv](https://github.com/GCY/wxECGAnalyzer/blob/master/res/snapshot.csv): example file.
    ![alt text](https://github.com/GCY/wxECGAnalyzer/blob/master/res/snapshot.png?raw=true) 
  - Validation of Algorithm for Real-time Embedded Systems(Holter)
    - 1.Add *.cpp and .h file, and create new wxRadioBox item to apply.
    - 2.Design algorithm with C/C++.
    - 3.Clinical trial and validation.
    - 4.Port code file to your embedded project.
    
    ![alt text](https://github.com/GCY/wxECGAnalyzer/blob/master/res/single%20mode.gif?raw=true) 
## Features
- [x] Filter
  - [x] Finite Impulse Response(FIR)
- [x] QRS-Complex Detect Algorithm
  - [x] Adaptive Threshold Algorithm
  - [x] HC_Chen Algorithm
  - [x] Enhanced So & Chen
  - [x] Pan-Tompkins
  - [ ] Deep-ECG(1D-CNN)
- [x] Heart Rate Variability
  - [x] Heart Rate
  - [x] SDNN
  - [x] NN50
  - [x] pNN50
- [x] Labeling
  - [x] Segmentation
  - [ ] Automatic segmentation
  - [x] MIT-BIH ECG-Codes
  - [ ] Automatic labeling(SVM+CNN Feature)
- [ ] MIT-BIH Database Operate
  - [ ] WFDB
- [ ] Quantitative
  - [ ] Sensitivity
  - [ ] Accuracy
  - [ ] Confusion Matrix
- [x] Other Operate  
  - [x] Record RAW data(60s)
  - [x] Save four plot(.png)  
- [x] Fast Furious Transform(FFT) amplitude spectrum
- [x] Connect serial port

## The point of QRS-Complex Detection Algorithm
### Finite Impulse Response
This project with FIR to filter ECG signal, coefficients generate parameter is 360hz, 32taps, band-pass 0.51Hz~8.9Hz and kaiser window.</br>
Coefficients Generator : https://github.com/GCY/Finite-Impulse-Response-FIR-Filter-
### Adaptive Threshold Algorithm
This algorithm purpose for this project, involving two parts, first is adaptive threshold update, and second find the local maxima and minima. </br>
Define threshold update period： (Sampling Rate / Target Low-Frequency), for example, target is ECG HR, normal people Heart Rate is 45-150 BPM, that is equally 0.75Hz-2.5Hz, 360SR/0.75Hz = 480 signal point, decrease update period the algorithm be sensitive.</br>
Determinate the local maxima and minima we need to know gradient, calculate below equation to find the gradient, Threshold_Factor for 12Bit ADC is 3.0f, increase Threshold_Factor, the local maximum and minimum are determinated by the algorithm which need more gradient.</br>
Gradient = RMS * CV(%) * Threshold_Factor</br>
Design more rules for the local maximum and minimum, your project will be able to recognize ECG PQRST.</br>
### HC Chen
This project modified LPF and HPF window size, hp_buffer = 150ms(QRS-Complex size), forgetting factor alpha is used to avoid threshold keeping high-level.</br>
### Enhanced So & Chen
The threshold_parameter and filter_parameter are 4 and 16 in the implementation , enhanced point is 123(350ms for 360Hz sampling rate), if last QRS-Complex point is over triple sampling rate, we will decrease threshold until zero, slope square process could detect abnormal heart rhythms.</br>
### Pan-Tompkins
The Pan-Tompkins filter of classical version is design for 200Hz and band-pass 5Hz-15Hz, if ECG signal is over 200Hz it needs to downsample to 200Hz, this project with FIR to clean 360Hz ECG signal, and modified Search Back period must be more than sampling_rate * 1.66(600 point, but this project is 550 point) .</br>
### Real-Time Complexity 
Running on STM32F407 clock 168MHz and enable FPU, y-axix time uint is nanoseconds, x-axix is signal point.</br>
Charts below show runtime environment time complexity, Adative Threshold Algorithm complexity is follow gradient threshold(step edge), the QRS-complex detection of the classical Pan-Tompkins algorithm mainly complexity is Search Back, HC Chen and So&Chen relatively stable.</br>
#### Adaptive Threshold Algorithm (Average : 8.100692259ns)
![alt text](https://github.com/GCY/wxECGAnalyzer/blob/master/res/ata%20time.png?raw=true)  
#### HC Chen (Average : 2.060941828ns)
![alt text](https://github.com/GCY/wxECGAnalyzer/blob/master/res/hc%20chen%20time.png?raw=true)  
#### Enhanced So & Chen (Average : 2.074ns)
![alt text](https://github.com/GCY/wxECGAnalyzer/blob/master/res/so%20and%20chen%20time.png?raw=true)  
#### Pan-Tompkins (Average : 548.0295567ns)
![alt text](https://github.com/GCY/wxECGAnalyzer/blob/master/res/pt%20time.png?raw=true)  
### Heart Rate Variability
Heart Rate and HRV are move-average in the implementation, N = BEAT_SIZE = 16.</br>
## Experiment device

### Devices
- ARM Cortex-M4
  - STM32F407-Discovery
  - AD8232
  
- [cNIBP](https://github.com/GCY/Continuous-Non-Invasive-Blood-Pressure-Research-Platform---ECG-and-PPG-Pulse-Arrival-Time-Based-.git): Use the ECG part.(without Right Leg Drive)

### Setup
Connect ECG signal output to STM32F4 PC0 pin, next load [*.elf](https://github.com/GCY/wxECGAnalyzer/tree/master/embedded) and run.
The setup ADC sampling rate is 360Hz with ADC + DMA + Timer-Trigger same as MIT-BIT arrhythmia database record.
</br></br>
For VCP mode just define</br>
#define VCP_MODE</br></br>
For Holter</br>
#define SINGLE_MODE</br>
And define QRS-Complex detect algorithm flag</br>
//#define ATA</br>
//#define HC</br>
//#define SO</br>
//#define PT</br>



## Video
</br>

### Algorithm test
[![Audi R8](http://img.youtube.com/vi/GpHpex1oun4/0.jpg)](https://youtu.be/GpHpex1oun4)

### Motion artifact - running
[![Audi R8](http://img.youtube.com/vi/KjEhhs2AV2s/0.jpg)](https://youtu.be/KjEhhs2AV2s)
### Disturbance
[![Audi R8](http://img.youtube.com/vi/AtoNvDiFkyU/0.jpg)](https://youtu.be/AtoNvDiFkyU)

</br>

## Reference
- Find the local maxima and minima : http://billauer.co.il/peakdet.html
- Heart rate variability : https://en.wikipedia.org/wiki/Heart_rate_variability
- HC_Chen Algorithm
  - HC Chen, SW Chen, "A Moving Average based Filtering System with its Application to Real-time QRS Detection", Computers in Cardiology, 2003.
  - https://github.com/blakeMilner/real_time_QRS_detection (modified)
- Enhanced So & Chen Algorithm
  - So, H. H., & Chan, K. L. (1997, October). Development of QRS detection method for real-time ambulatory cardiac monitor. In Proceedings of the 19th Annual International Conference of the IEEE Engineering in Medicine and Biology Society.'Magnificent Milestones and Emerging Opportunities in Medical Engineering'(Cat. No. 97CH36136) (Vol. 1, pp. 289-292). IEEE.
  - Lee, R. G., Chou, I. C., Lai, C. C., Liu, M. H., & Chiu, M. J. (2005). A novel QRS detection algorithm applied to the analysis for heart rate variability of patients with sleep apnea. Biomedical Engineering: Applications, Basis and Communications, 17(05), 258-262.
  - Wang, C. Y., Chang, R. C. H., Lin, C. H., & Su, S. H. (2018, May). Fatigue Detection System Using Enhanced So and Chan Method. In 2018 IEEE International Conference on Consumer Electronics-Taiwan (ICCE-TW) (pp. 1-2). IEEE.
- Pan-Tompkins Algorithm
  - Pan, J., & Tompkins, W. J. (1985). A real-time QRS detection algorithm. IEEE transactions on biomedical engineering, (3), 230-236.
  - Afonso, V. X. (1993). Ecg qrs detection. Biomedical digital signal processing, 237-264.
  - https://github.com/rafaelmmoreira/PanTompkinsQRS (modified)

LICENSE
-------

MIT License

Copyright (c) 2019 Tony Guo

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
