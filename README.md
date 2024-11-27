# LAB-6-OFDM
clc;
clear;
close all;
n = 256;
M = 16;
k = log2(M);
nmax = 64;
scb = 312.5e3;
fc = 3.6e9;
tu = 3.2e-6;
tg = 0.8e-6;
ts = tu + tg;
snr = 10;
X = randi([0, 1], n, 1);
xsym = bi2de(reshape(X, k, length(X)/k).', 'left-msb');
y = qammod(xsym, M);
tt = 0:6.25e-8:ts-6.25e-8;
time_ofdm = linspace(0, ts, nmax);
c = ifft(y, nmax);
s = real(c .* exp(1j * 2 * pi * fc * time_ofdm));
% Create a single figure with multiple subplots
figure;
% Subplot 1: Transmitted OFDM Signal
subplot(2, 2, 1);
plot(real(s), 'b');
title('OFDM Signal Transmitted');
xlabel('Time (s)');
ylabel('Amplitude');
% Subplot 2: Power Spectral Density of the Transmitted Signal
subplot(2, 2, 2);
plot(10*log10(abs(fftshift(fft(s, nmax)))), 'r');
xlabel('Frequency (Hz)');
ylabel('Power Spectral Density (dB)');
title('Transmit Spectrum of OFDM');
% Add AWGN noise to the signal
ynoisy = awgn(s, snr, 'measured');
% Subplot 3: Received Signal with Noise
subplot(2, 2, 3);
plot(real(ynoisy), 'b');
title('Received OFDM Signal with Noise');
xlabel('Time (s)');
ylabel('Amplitude');
% Perform bandpass demodulation and FFT
z = ynoisy .* exp(-1j * 2 * pi * fc * time_ofdm);
z = fft(z, nmax);
zsym = qamdemod(z, M);
zbin = de2bi(zsym, k, 'left-msb');
zbin = zbin(:); % Flatten the binary matrix
% Compare the original bits with the demodulated bits
[noe, ber] = biterr(X, zbin(1:length(X)));
% Display BER information
disp(['Number of Bit Errors: ', num2str(noe)]);
disp(['Bit Error Rate (BER): ', num2str(ber)]);
% Subplot 4: Original vs. Recovered Message
subplot(2, 2, 4);
stem(X(1:256), 'r', 'Marker', 'none');
hold on;
stem(zbin(1:256), 'b', 'Marker', 'none');
title('Original and Recovered Message');
legend('Original Message', 'Recovered Message');
xlabel('Bit Index');
ylabel('Bit Value');
