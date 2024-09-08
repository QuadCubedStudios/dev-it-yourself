# .aiff: Audio Interchange File Format
While not a common audio format in the modern era, AIFF is relatively simple compared to popular formats such as MP3, OGG Theora or FLAC because it can store uncompressed audio data. Use the description of the format below, and write a program that generates an audio file.

## A basic intro to Digital Audio
When dealing with analog audio, you generally treat it as a continuous waveform measured at a given point in space, where the amplitude represents the relative pressure level at that point. The red line below shows an example of an analog waveform:

![Sound wave with PCM](https://upload.wikimedia.org/wikipedia/commons/2/21/4-bit-linear-PCM.svg)

However, you would need an infinite amount of information to store the exact waveform in digital form, as there are an infinite number of time and pressure values to be stored. Since this is prohibitive for a digital system, we need to quantize it.

A very common approach to quantize audio is PCM (Pulse Code Modulation), shown in the blue dots above. The scheme boils down to measuring the amplitude of the signal at discrete time intervals, and further discretizing the measured amplitude into a signed binary number.

The number of divisions in the time dimension is the *sample rate* of the scheme, and the number of bits required to represent all possible amplitude values is the *bit depth*.

In the above example, if we assume the wave is taken over 1 millisecond, there are 23 samples across that interval which gives us a sample rate of **23,000 Hz**. And since it takes 4 bits to represent the 16 amplitude values (-8 to +7), it has a bit-depth of **4**.

## Structure of an .aiff file
To simplify things, we will work with the original AIFF standard (more details can be found [here](https://www-mmsp.ece.mcgill.ca/Documents/AudioFormats/AIFF/Docs/AIFF-1.3.pdf)).

**NOTE**: This format was defined by Apple when they were using the Motorola 68000 as their CPU, which operated with Big Endian values. Hence, all multi-byte values in this format are **Big Endian** in nature.

AIFF files are raw binary files, built up using "chunks" that have an ID and data:
```c
Chunk {
    // ID stands for char[4], and represents a text word that identifies the chunk

    ID      chunk_id;       // The type of this chunk
    i32     chunk_size;     // Number of bytes taken up by the chunk_data section

    u8[]    chunk_data;     // The data stored in this chunk
}
```

The whole .aiff file is made up of one such "container" chunk called a *Form*, which stores other chunks:
```c
AiffFormChunk {
    // These are fields from the generic Chunk type
    ID      chunk_id;       // Must be set to "FORM"
    i32     chunk_size;     // This is set to the total size of form_type and chunks (in bytes)

    // These two below form the `chunk_data` of the generic Chunk
    ID      form_type;      // Must be set to "AIFF"
    Chunk[] chunks;         // All sub-chunks in this container
}
```

There are two mandatory chunks for a valid AIFF file: the Common chunk, and the Sound Data chunk.

### Common chunk
Programs require metadata to understand how to read the audio stored in the Sound Data chunk:
```c
CommonChunk {
    // These are fields from the generic Chunk type
    ID      chunk_id;           // Must be set to "COMM"
    i32     chunk_size;         // This is set to 18: (16 + 32 + 16 + 80) / 8

    // These form the `chunk_data` of the generic Chunk
    i16     num_channels;       // Number of audio channels
    u32     num_sample_frames;  // Number of Sample Frames
    i16     sample_size;        // Bit depth of each sample point
    f80     sample_rate;        // Number of sample frames per second
}
```

Since the format is capable of handling multi-channel audio, instead of storing a single amplitude value per point in time, we store one amplitude per channel. The spec calls this a "Sample Frame", with these parameters in the Common Chunk describing the structure of these frames:

1. `num_channels`: This is the number of channels in each frame. Examples are Mono (1 channel), Stereo (2 channels) or Surround (3+ channels) sound.
2. `num_sample_frames`: This is the sample rate of the stored sound in terms of frames per second.
3. `sample_size`: This is the bit-depth of each sample point within a frame. It's best to store 8, 16 or 32 here, since these form integer multiples of a byte.

### Sound Data chunk
The actual PCM audio is stored here:
```c
SoundDataChunk {
    // These are fields from the generic Chunk type
    ID      chunk_id;       // Must be set to "SSND"
    i32     chunk_size;     // This will be 4 + 4 + the length of the sample_frames array

    // These form the `chunk_data` of the generic Chunk
    u32     offset;         // How many bytes to skip in sample_frames before reaching the first frame.
    u32     block_size;     // Number of Sample Frames in a block of audio
    Frame[] sample_frames;  // The actual Sample Frames of audio
}
```

For most applications both the `offset` and `block_size` will be set to 0. `sample_frames` contains the bytes for the sample frames making up the actual audio.

Depending on the bit-depth you choose, each sample point will be an `i8`, `i16` or `i32`. A `Frame` is then composed of `num_channel` sample points stored contiguously. There is a convention on how the sample points within a frame are ordered, as follows (change `i16` to whichever bit-depth you choose):
```
MonoFrame {
    i16 sample;
}

StereoFrame {
    i16 left;
    i16 right;
}

ThreeChannelFrame {
    i16 left;
    i16 right;
    i16 center;
}
```
