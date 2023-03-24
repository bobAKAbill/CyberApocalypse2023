# Cyber Apocalypse 2023

## Analogue Signal Processing

> Before the aliens had computers, they used analogue circuits to communicate. One of their favourite tricks was to hide messages in the communication equipment itself, which often used inductors and high-tech, complex resistors. Using a few samples from different locations in the circuit, can you figure out the parameters of the circuitry?
>
>  Readme Author: TheBadGod provided the code, [bobAKAbill](github.com/bobakabill) wrote the rest
> THE ZIP WAS TOO LARGE TO PUT INTO GITHUB
> [`rev_analogue_signal_processing.zip`](https://drive.google.com/file/d/11ggbYH_GbZDY1CtsQ4i7T1v_xwgmxLpR/view?usp=sharing)

Download the zip, put the below python next to `circuit.py`, run the python, wait

```
import numpy as np
from circuit import ZLCircuit, simulate_chained_circuits
import soundfile as sf
import sys

duration = 30
SAMPLE_RATE = 0x4000

samples = int(duration * SAMPLE_RATE)

try:
    vins, _ = sf.read("vins.wav", dtype="float64")
except:
    s0, _ = sf.read(f"./audio/encoded0.wav", dtype="float64")
    s0 = s0.astype("complex128")
    h = 1. / SAMPLE_RATE
    for c in range(0x20, 0x80):
        print("Trying character:", chr(c))
        try:
            c = float(c)
            # create v_in array
            vins = [0]*(SAMPLE_RATE*duration)
            # state is 0 initially
            si = 0+0j
            for idx in range(SAMPLE_RATE * duration):
                # calculate initial state of V_in[j], im(V_in) = 0 due to initialization
                x = s0[idx].real - si.real - h * (1.0 - h*h*c*c/6.0) * c * si.imag
                x /= h * (h*h*h*c*c*c / 24.0 - h*c / 2.0) * c
                x *= -1
                x += si.real

                # all vin drawn at random from (-1, 1)
                assert -1 <= x <= 1

                # use normal runge-kutta to calculate next si
                A = (0-1j)*c
                K = ((A*h)**3 + 4 * (A*h)**2 + 12 * (A*h) + 24) * h / 24
                #si2 = si + K * A * (si - x)

                Xn = 1.0 + K * A
                Bu = K * x * -A
                si = Xn * si + Bu

                vins[idx] = x
        except KeyboardInterrupt:
            exit(1)
        except:
            continue

        vins = np.asarray(vins)
        sf.write("vins.wav", vins, SAMPLE_RATE)

        print("RAN THROUGH"*100)
        vout = simulate_chained_circuits([ZLCircuit(1j * c, 1)], vins, duration, SAMPLE_RATE)[0].real
        s = sum([abs(i.real-j.real) for i, j in zip(s0, vout)])
        print(c, s)
        break


flag = ""
idx = 0
while True:
    s1, _ = sf.read(f"./audio/encoded{idx}.wav", dtype="float64")
    s1 = s1.astype("complex128")
    idx += 1
    l = []
    for c in b"abcdefghijklmnopqrstuvwxyz_0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ@{}":
        vout = simulate_chained_circuits([ZLCircuit(1j * c, 1)], np.array(vins, copy=True), duration, SAMPLE_RATE)[0].real
        s = sum([abs(i-j) for i, j in zip(s1, vout)])
        l.append((s, chr(c)))
        print(idx, flag, ":", sorted(l, key=lambda x: x[0])[:3])
    print(idx, flag, ":", sorted(l, key=lambda x: x[0])[:3])
    flag += sorted(l, key=lambda x: x[0])[0][1]
    # actually apply the last char
    vins = simulate_chained_circuits([ZLCircuit(1j * ord(flag[-1]), 1)], np.array(vins, copy=True), duration, SAMPLE_RATE)[0]
```
## Flag
HTB{p0le_dance}