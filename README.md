# hello-world
Just introduction of git-hub

Hi, I'm Ingyu Jang!


import numpy as np
from matplotlib import pyplot as plt

c = 3e8                                 # light speed
wavelength = 1550e-9                    # wavelength
del_wavelength = 2e-9                   # wavelength range
f1 = c / wavelength                     # max frequency
f0 = c / (wavelength + del_wavelength)  # min frequency
T = 2e-3                                # period
distance = 250                          # target distance
tau = distance / c                      # target distance / light speed
r = 0.1                                 # target reflectance
E = 1                                   # light power
fs = 500e6                              # sampling frequency

time = np.arange(0, T, 1 / fs)
time_length = len(time)

Emission = E * np.exp(1j * 2 * np.pi * (f0 + (f1 - f0) / T * time) * time)
Reflection = r * E * np.exp(1j * 2 * np.pi * (f0 + (f1 - f0) / T * (time - tau)) * (time - tau))

Addition = Emission + Reflection

Intensity = np.abs(Addition)

fft_result = np.fft.fft(Intensity)
fft_result = fft_result[0:int(time_length / 2)]
frequency_range = fs * np.arange(0, int(time_length / 2))

plt.plot(frequency_range[1:], abs(fft_result[1:]))
plt.show()
