# WAVELET-AUDIO-DENOISING
WAVELET AUDIO DENOISING
>> clc
close all
clear all
% De-Noising Audio Signal Using Discrete Wavelet Transform
%-------------------------------------------------------------
% The de-noising procedure proceeds in three steps:
% [1] Decomposition
% Choose a wavelet, and choose a level N. Compute
% the wavelet decomposition of the signal s at level N
% [2] Detail coefficients thresholding
% For each level from 1 to N, select a threshold
% and apply soft thresholding to the detail coefficients
% [3] Reconstruction
% Compute wavelet reconstruction based on the original
% approximation coefficients of level N and the modified
% detail coefficients of levels from 1 to N
%-------------------------------------------------------------
fprintf('--- De-noising Audio Signal Using Discrete Wavelet Transform ---\n\n');
%-------------------------%
% load example of sound %
%-------------------------%
fprintf('\t\tLoad "chirp.wav" - ');
% [ipsignal, Fs] = audioread('chirp.wav');
[ipsignal, Fs] = audioread('seiska.wav');
% ipsignal = ipsigna(1:length(ipsigna)/2);
amp = 20;
ipsignal = amp*ipsignal;
N = length(ipsignal);
fprintf('OK\n');
%----------------------------%
% add white gaussian noise %
%----------------------------%
fprintf('\t\tAdd White Gaussian noise - ');
%the scalar SNR specifies the signal-to-noise ratio per sample, in dB
sn = 5;
%add white gaussian noise to a signal
ipsignalN = awgn(ipsignal,sn);
fprintf('OK\n\n');
%---------------------------%
% DWT %
% wavelet decomposition %
%---------------------------%
fprintf('Step 1: Decompose Audio Signal - ');
level = 3;
fprintf('\n\t\tChoose Wavelet:\n\t\t\t(1) Daubechies-13\n\t\t\t(2) Daubechies-40\n\t\t\t(3) Symlet-13\n\t\t\t(4) Symlet-21\n\t\t');
wname = input('Enter your choice : ');
if wname == 1
wt = 'db13';
elseif wname == 2
wt = 'db40';
elseif wname == 3
wt = 'sym13';
elseif wname == 4
wt = 'sym21';
end
%computes four filters
[Lo_D,Hi_D,Lo_R,Hi_R] = wfilters(wt);
[C,L] = wavedec(ipsignalN,level,Lo_D,Hi_D);
cA3 = appcoef(C,L,wt,level);
%extract the levels 3, 2, and 1 detail coefficients from C
[cD1,cD2,cD3] = detcoef(C,L,[1,2,3]);
%reconstruct the level 3 approximation from C
A3 = wrcoef('a',C,L,Lo_R,Hi_R,level);
%reconstruct the details at levels 1, 2, and 3, from C
D1 = wrcoef('d',C,L,Lo_R,Hi_R,1);
D2 = wrcoef('d',C,L,Lo_R,Hi_R,2);
D3 = wrcoef('d',C,L,Lo_R,Hi_R,3);
%a = approximation
%d = detail
%---------------------------%
% thresholding %
%---------------------------%
fprintf('\nStep 2: Thresholding - ');
fprintf('\n\t\tChoose Threshold Rule:\n\t\t\t(1) Universal\n\t\t\t(2) Minimax\n\t\t');
tr = input('Enter your choice : ');
if tr == 1
tptr = 'sqtwolog';
elseif tr == 2
tptr = 'minimaxi';
end
thr_D1 = thselect(D1,tptr);
thr_D2 = thselect(D2,tptr);
thr_D3 = thselect(D3,tptr);
fprintf('\n\t\tChoose Threshold Type:\n\t\t\t(1) Soft\n\t\t\t(2) Hard\n\t\t');
sh = input('Enter your choice : ');
if sh == 1
sorh = 's';
elseif sh == 2
sorh = 'h';
end
fprintf('\n\t\tChosen Parameters : %s, %s, %s\n\n',wt,tptr,sorh);
%threshold coefficient of details
tD1 = wthresh(D1,sorh,thr_D1);
tD2 = wthresh(D2,sorh,thr_D2);
tD3 = wthresh(D3,sorh,thr_D3);
%--------------------------%
% compute Inverse DWT %
%--------------------------%
fprintf('Step 3: Compute Inverse DWT');
denoised = A3 + tD1 + tD2 + tD3;
err = max(abs(ipsignalN-denoised));
%---------------------------%
% compute SNR %
%---------------------------%
fprintf('\n\t\tCompute SNR\n');
%SNR - Signal to Noise Ratio
SNR = snr(ipsignal,ipsignalN);
NoisySNR = 20*log10(norm(ipsignal(:)) / norm (ipsignal(:)-ipsignalN(:)) )
dSNR = snr(ipsignal,denoised);
DenoisedSNR = 20*log10(norm(ipsignal(:)) / norm (ipsignal(:)-denoised(:)) )
% ----------------- Listen Result -----------------------------------
% fprintf('\n\t\tPlaying the Original Sound:');
% play1 = audioplayer(ipsignal,Fs);
% playblocking(play1);
% fprintf(' OK');
% fprintf('\n\t\tPlaying the Noisy Sound:');
% play2 = audioplayer(ipsignalN,Fs);
% playblocking(play2);
% fprintf(' OK');
% fprintf('\n\t\tPlaying the Denoised Sound:');
% play3 = audioplayer(denoised,Fs);
% playblocking(play3);
% fprintf(' OK\n');
% ----------------- Display Figure ----------------------------------
figure
subplot(3,1,1); plot(ipsignal); title('True Speech Signal');
xlabel('Samples'); ylabel('Amplitude');
subplot(3,1,2); plot(ipsignalN); title('Noisy Speech Signal');
xlabel('Samples'); ylabel('Amplitude');
subplot(3,1,3); plot(denoised); title('De-noised Speech Signal');
xlabel('Samples'); ylabel('Amplitude');
