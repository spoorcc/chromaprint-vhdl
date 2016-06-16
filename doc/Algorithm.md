High-level steps
================================================================================


Based on [this](https://oxygene.sk/2011/01/how-does-chromaprint-work/)

- Make mono if needed
- Downsample to 11025 Hz
- FFT
- Extract chroma-features (12 bins + filtering + normalization)
- Create fingerprint stream using sliding window with 16 filters
- Quantize each filter
- Encode gray code of each filter in 2 bits/filter, total 32 bits

More details from [source code](https://github.com/acoustid/chromaprint/tree/master/src)

AudioProcessor
--------------------------------------------------------------------------------

Takes in several types of audio data, converts it to the following MONO stream.

     TYPE = uint
     WORDSIZE = 16
     SAMPLE_RATE = 11025

FFT
--------------------------------------------------------------------------------

Transforms audio from the AudioProcessor into the frequency domain.


### Hamming Window ###

A standard Hamming window is used for smoothing:

    ALPHA = 0.54
    BETA = 0.46
    N = FFT_FRAME_SIZE

    w(n) = ALPHA - BETA * cos(2*pi*n / (N - 1))

See also [wikipedia](https://en.wikipedia.org/wiki/Window_function#Hamming_window)

### Actual FFT ###

A 4096 point FFT is taken from the windowed frame, after which the real + img parts
are squared and summed (but root is not taken). This FFT is done with 2/3 overlap.

Settings in FFT:

     FFT_FRAME_SIZE = 4096
     SAMPLE_RATE = 11025
     OVERLAP = FRAME_SIZE - FRAME_SIZE / 3;// 2720;
     STEP_LENGTH = FFT_FRAME_SIZE / SAMPLE_RATE / OVERLAP



     WINDOW_SIZE = 16
     FILTER_SIZE = 5
     FP_SIZE = (LENGTH - STEP_LENGTH * (WINDOW_SIZE + FILTER_SIZE)) / STEP_LENGTH

Chroma
--------------------------------------------------------------------------------

### Prepare Notes ###

In order to place all the frequencies in the correct bins for each of the
points in the FFT the bin index is precalculated.

      NUM_OF_BANDS = 12
      MIN_FREQ = 28
      MAX_FREQ = 3520
      FFT_FRAME_SIZE = 4096
      SAMPLE_RATE = 11025

      BASE = 440.0 Hz / 16.0

      minimum_index = max(1, (int)round(FFT_FRAME_SIZE * MIN_FREQ / SAMPLE_RATE))
      max_index     = min(FFT_FRAME_SIZE/2, (int)round(FFT_FRAME_SIZE * MAX_FREQ / SAMPLE_RATE))

      for index in range(minimum_index, max_index):
          freq = index * SAMPLE_RATE / FFT_FRAME_SIZE
          octave = log(freq / base) / log(2.0)
          note = NUM_OF_BANDS * (octave - floor(octave))
          notes[index] = note

### Extract Chroma features ###
There is an interpolated and uninterpolated version

#### Without interpolation ####

Without interpolation it is trivial:

    for index, energy in fft_frame:
        feature_bins[pre_calculated_index[index]] += energy

#### With interpolation ####

TODO

ChromaFilter
--------------------------------------------------------------------------------
The 12 feature bins are filtered?

       kChromaFilterSize = 5;
       kChromaFilterCoefficients[] = { 0.25, 0.75, 1.0, 0.75, 0.25 };

ChromaNormalizer
--------------------------------------------------------------------------------
Normalize the vector by calculating euclidean norm (square the sum of squares)
If this is above noise threshold (0.01) divide each feature by the norm.


ImageBuilder
--------------------------------------------------------------------------------
Add each normalized vector as new column to the image


FingerprintCalculator
--------------------------------------------------------------------------------

### Create Integral image ###
Image transformation that allows us to quickly calculate the sum of values in a rectangular area. [summed area table](http://en.wikipedia.org/wiki/Summed_area_table)

### Classify image ###
Using multiple pre-trained classifiers, classify the image. Each classifier consists
of a filter and a quantizer.

#### Filter image using predefined filters ####
First apply one of 5 filters which are described by x,y width and height

Each of the 5 filters have black and white areas in which they are subtracted from eachoter using a logarithmic function:

    result = log(1 + a) - log(1 + b)

#### Quantize filter result ####
The outcome of the filter stage is Quantized to 0-3 range. The thresholds for
quantization are precalculated and together with the filter, they form a classifier.
Three threshold values determine how it is quantized.

    0---->t1---->t2---->t3---->max

### Convert Quantized result to gray code ###
Gray code encodes the 2 bits in continuous bits

    00 = 0
    01 = 1
    11 = 2
    10 = 3



