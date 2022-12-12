# Radar_Target_Generation_and_Detection
## 1.FMCW Waveform Design
**Criteria:** 

Using the given system requirements, design a FMCW waveform. Find its Bandwidth (B), chirp time (Tchirp) and slope of the chirp.

**MEETS SPECIFICATIONS:**  

For given system requirements the calculated slope should be around 2e13 

**My Code**
 ```matlab
   %% Radar Specifications 
   %%%%%%%%%%%%%%%%%%%%%%%%%%%
   % Frequency of operation = 77GHz
   % Max Range = 200m
   % Range Resolution = 1 m
   % Max Velocity = 100 m/s
   %%%%%%%%%%%%%%%%%%%%%%%%%%%
   B=3e8/(2*1);
   Tchirp=5.5*2*200/3e8;
   slope=B/Tchirp;
   ```
**Result**
 ```matlab
   >> slope

   slope =

      2.0455e+13
 ```
