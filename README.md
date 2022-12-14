# Radar_Target_Generation_and_Detection
## 1. FMCW Waveform Design
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
## 2. FMCW Waveform Design
**Criteria:** 

Simulate Target movement and calculate the beat or mixed signal for every timestamp.

**MEETS SPECIFICATIONS:**  

A beat signal should be generated such that once range FFT implemented, it gives the correct range i.e the initial position of target assigned with an error margin of +/- 10 meters.

**My Code**
 ```matlab
   for i=1:length(t)         
    
    
    % *%TODO* :
    %For each time stamp update the Range of the Target for constant velocity. 
    R_cur=R+v*t(i);
    tao=2*R_cur/3e8;
    % *%TODO* :
    %For each time sample we need update the transmitted and
    %received signal. 
    alpha=slope;
    Tx(i) =  cos(2*pi*(fc*t(i)+alpha*t(i)^2/2));
    Rx(i) =  cos(2*pi*(fc*(t(i)-tao)+alpha*(t(i)-tao)^2/2));
    
    % *%TODO* :
    %Now by mixing the Transmit and Receive generate the beat signal
    %This is done by element wise matrix multiplication of Transmit and
    %Receiver Signal
    %Mix(i) =cos(2*pi*(2*alpha*R/3e8*t(i)+2*fc*v/3e8*t(i)));
    Mix(i)=Tx(i).* Rx(i);
   end
   ```


## 3. FMCW Waveform Design
**Criteria:** 

Implement the Range FFT on the Beat or Mixed Signal and plot the result.

**MEETS SPECIFICATIONS:**  

A correct implementation should generate a peak at the correct range, i.e the
initial position of target assigned with an error margin of +/- 10 meters.

**My Code**
 ```matlab
   Mix_2d = reshape(Mix, [Nr, Nd]);
   % *%TODO* :
   %run the FFT on the beat signal along the range bins dimension (Nr) and
   %normalize.
   X=Mix_2d(:,1);
   signal_fft1 = fft(X)./Nr;
   % *%TODO* :
   % Take the absolute value of FFT output
   signal_fft2 = abs(signal_fft1(1:Nr));
   % *%TODO* :
   % Output of FFT is double sided signal, but we are interested in only one side of the spectrum.
   % Hence we throw out half of the samples.

   signal_fft3  = signal_fft2;       

   %plotting the range
   figure ('Name','Range from First FFT')
   subplot(2,1,1)

   % *%TODO* :
   % plot FFT output 
   plot(signal_fft3);

   axis ([0 200 0 0.5]);
   ```
**Result**

![FFT1D](FFT1D.jpg)

## 4. FMCW Waveform Design
**Criteria:** 

1.Implement the 2D CFAR process on the output of 2D FFT operation, i.e the Range Doppler Map.

2.Create a CFAR README File

**MEETS SPECIFICATIONS:**  

1.The 2D CFAR processing should be able to suppress the noise and separate the target signal. The output should match the image shared in walkthrough.
2.In a README file, write brief explanations for the following:
- Implementation steps for the 2D CFAR process.
- Selection of Training, Guard cells and offset.
- Steps taken to suppress the non-thresholded cells at the edges.

**Aproach:**  
The 2D CFAR is similar to 1D CFAR, but is implemented in both dimensions of the range doppler block. The 2D CA-CFAR implementation involves the training cells occupying the cells surrounding the cell under test with a guard grid in between to prevent the impact of a target signal on the noise estimate.

   1. Select the number of Training Cells and Guard Cells in both the dimensions and set offset of threshold
   ```matlab
   % *%TODO* :
   %Select the number of Training Cells in both the dimensions.
   Tr=10;
   Td=8;
   % *%TODO* :
   %Select the number of Guard Cells in both dimensions around the Cell under 
   %test (CUT) for accurate estimation
   Gr=4;
   Gd=4;
   % *%TODO* :
   % offset the threshold by SNR value in dB
   off_set=1.4;
   ```
   2. Slide Window through the complete Range Doppler Map
   ```matlab
     RDM_temp=db2pow(RDM);
     final_data=zeros(Nr/2,Nd);
     initial_data=zeros(Nr/2-(2*(Gr+Tr)),Nd-(2*(Gd+Td)));
     for i = 1:(Nr/2-(2*(Gr+Tr)+1)) 
         for j=1:(Nd-(2*(Gd+Td)+1))
             data_all=RDM_temp(i:i+2*(Tr+Gr)+1,j:j+2*(Td+Gd)+1);
             data_inner=RDM_temp(i+Tr:i+Tr+2*Gr+1,j+Td:j+Td+2*Gd+1);
             sum_tain=sum(sum(data_all))-sum(sum(data_inner));
             train_number=(2*(Gr+Tr)+1)*(2*(Gd+Td)+1)-(2*Gr+1)*(2*Gd+1);
             mean_train=sum_tain/train_number;
             mean_dbd=pow2db(mean_train);
             thres=mean_dbd*off_set;
             if(RDM(i+Tr+Gr,j+Td+Gd)>thres)
                  initial_data(i,j)=1;
             else
                  initial_data(i,j)=0;
             end
         end      
     end
   ```
   3. Steps taken to suppress the non-thresholded cells at the edges.
   ```matlab
     % *%TODO* :
     % The process above will generate a thresholded block, which is smaller 
     %than the Range Doppler Map as the CUT cannot be located at the edges of
     %matrix. Hence,few cells will not be thresholded. To keep the map size same
     % set those values to 0. 
     
     final_data(Gr+Tr+1:Nr/2-(Gr+Tr),Gd+Td+1:Nd-(Gd+Td))=initial_data;
    
   ```
**Final Result:**  
![CA-CFAR](CA-CFAR.jpg)
