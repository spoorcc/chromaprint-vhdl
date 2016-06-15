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

Mono
--------------------------------------------------------------------------------

Downsample
--------------------------------------------------------------------------------

     TYPE = uint
     WORDSIZE = 16
     SAMPLE_RATE = 11025

FFT
--------------------------------------------------------------------------------

Settings in FFT:

     FFT_FRAME_SIZE = 4096
     SAMPLE_RATE = 11025
     OVERLAP = 3
     STEP_LENGTH = FFT_FRAME_SIZE / SAMPLE_RATE / OVERLAP
     WINDOW_SIZE = 16
     FILTER_SIZE = 5
     FP_SIZE = (LENGTH - STEP_LENGTH * (WINDOW_SIZE + FILTER_SIZE)) / STEP_LENGTH

Extract Chroma Features
--------------------------------------------------------------------------------


Filter image using predefined filters
--------------------------------------------------------------------------------


Quantize each filter outcome to 0-3 range
--------------------------------------------------------------------------------


Convert Quantized result to gray code
--------------------------------------------------------------------------------

