# Useful Audio DSP Algorithms

## Rules of Logarithm

```
log(x * y) = log(x) + log(y)
log(x / y) = log(x) - log(y)
log(x ^ y) = y * log(x)
log(1 / x) = -log(x)
log(1) = 0
NOT for log10: ln(e) = 1
```

## Decibels / amplitude conversion:

```eel
decibels = 20 * log(abs(amplitude));
amplitude = 10 ^ (decibels / 20);
```

## Inifinite Impulse Response Filter (IIRF)

Tutorial: https://www.youtube.com/watch?v=HexzW0EZal8

```eel
@sample
average = (spl0 + previous) / 2;
previous = spl0;
spl0 = spl1 = average;
```

### Biquad Filters

https://www.w3.org/TR/audio-eq-cookbook/

#### Intermediate Variables

```eel
A = 10 ^ (gain / 40);               // Only for peaking & shelving EQs
omega = 2 * $pi * freq / srate;
sinOmega = sin(omega);
cosOmega = cos(omega);
alpha = sinOmega / (2 * q);
```

#### Coefficients Per Filter Type

```eel
// High Pass

b0 = (1 + cosOmega) / 2;
b1 = - (1 + cosOmega);
b2 = (1 + cosOmega) / 2;
a0 = 1 + alpha;
a1 = -2 \* cosOmega;
a2 = 1 - alpha;

// Low Pass

b0 = (1 - cosOmega) / 2;
b1 = 1 - cosOmega;
b2 = (1 - cosOmega) / 2;
a0 = 1 + alpha;
a1 = -2 \* cosOmega;
a2 = 1 - alpha;

// Low Shelf

b0 = A * ((A + 1) - (A - 1) * cosOmega + 2 * sqrt(A) * alpha);
b1 = 2 * A * ((A - 1) - (A + 1) * cosOmega);
b2 = A * ((A + 1) - (A - 1) * cosOmega - 2 * sqrt(A) * alpha);
a0 = (A + 1) + (A - 1) * cosOmega + 2 * sqrt(A) * alpha;
a1 = -2 * ((A - 1) + (A + 1) * cosOmega);
a2 = (A + 1) + (A - 1) * cosOmega - 2 * sqrt(A) * alpha;

// High Shelf

b0 = A * ((A + 1) + (A - 1) * cosOmega + 2 * sqrt(A) * alpha);
b1 = -2 * A * ((A - 1) + (A + 1) * cosOmega);
b2 = A * ((A + 1) + (A - 1) * cosOmega - 2 * sqrt(A) * alpha);
a0 = (A + 1) - (A - 1) * cosOmega + 2 * sqrt(A) * alpha;
a1 = 2 * ((A - 1) - (A + 1) * cosOmega);
a2 = (A + 1) - (A - 1) * cosOmega - 2 * sqrt(A) * alpha;

// Peak EQ

b0 = 1 + alpha * A;
b1 = -2 * cosOmega;
b2 = 1 - alpha * A;
a0 = 1 + alpha / A;
a1 = -2 * cosOmega;
a2 = 1 - alpha / A;
```

#### The Transfer Function

```eel
@sample

y2 = y1;
y1 = y;
y = (b0 / a0) * x
    + (b1 / a0) * x1
    + (b2 / a0) * x2
    - (a1 / a0) * y1
    - (a2 / a0) * y2;

x2 = x1;
x1 = x;
x = spl0;       // The current sample

spl0 = y;       // y: Our resulting value
```