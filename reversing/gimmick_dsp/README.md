# Cyber Apocalypse 2023

## Gimmick DSP

> Before the aliens had computers, they used analogue circuits to communicate. One of their favourite tricks was to hide messages in the communication equipment itself, which often used inductors and high-tech, complex resistors. Using a few samples from different locations in the circuit, can you figure out the parameters of the circuitry?
>
>  Readme Author: Om012 provided the code, [bobAKAbill](github.com/bobakabill) wrote the rest
>
> [`rev_gimmick_dsp.zip`](rev_gimmick_dsp.zip)

Download the flag, run the python, listen

```
import librosa
import numpy as np
import soundfile as sf
from scipy import signal

# load the audio file
signal_data, sample_rate = sf.read('output.wav')

# apply a band-stop filter to remove the hiss and hum
nyquist_freq = 0.5 * sample_rate
low_freq = 50
high_freq = 1000
order = 4
b, a = signal.butter(order, [low_freq/nyquist_freq, high_freq/nyquist_freq], btype='bandstop')
filtered_signal = signal.lfilter(b, a, signal_data)

# calculate the spectrogram of the filtered signal
n_fft = 2048
hop_length = 512
win_length = 2048
window = 'hann'
spectrogram = librosa.stft(filtered_signal, n_fft=n_fft, hop_length=hop_length, win_length=win_length, window=window)
spectrogram = np.abs(spectrogram) ** 2

# estimate the spectral centroid of the noise
spectral_centroid = librosa.feature.spectral_centroid(S=spectrogram, sr=sample_rate, n_fft=n_fft, hop_length=hop_length)
spectral_noise_centroid = np.median(spectral_centroid)

# apply spectral subtraction to the spectrogram
eps = 1e-8
spectral_subtraction_mask = 1.0 - np.sqrt(spectral_noise_centroid / (spectrogram.T + eps))
spectral_subtraction_mask = np.maximum(spectral_subtraction_mask, 0.0)
spectral_subtracted = np.multiply(spectrogram.T, spectral_subtraction_mask)
spectral_subtracted = np.sqrt(spectral_subtracted)
spectral_subtracted = librosa.istft(spectral_subtracted.T, hop_length=hop_length, win_length=win_length, window=window)

# zero-pad the spectrogram to match the length of the filtered signal
n_samples = len(filtered_signal)
spectral_subtracted_padded = np.zeros(n_samples)
spectral_subtracted_padded[:len(spectral_subtracted)] = spectral_subtracted

# subtract the noise from the filtered signal
filtered_signal -= spectral_subtracted_padded


# save the output
sf.write('flag.wav', filtered_signal, sample_rate)
```