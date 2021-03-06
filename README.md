# GIF en- and decoding library [![Build Status](https://travis-ci.org/PistonDevelopers/image-gif.svg?branch=master)](https://travis-ci.org/PistonDevelopers/image-gif)

GIF en- and decoder written in Rust ([API Documentation](http://www.piston.rs/image/gif/index.html)). 

# GIF encoding and decoding library

This library provides all functions necessary to de- and encode GIF files. 

## High level interface

The high level interface consists of the two types
Encoder and Decoder.
They as builders for the actual en- and decoders and can be used to set various
options beforehand.

### Decoding GIF files

```
// Open the file
use std::fs::File;
use gif::SetParameter;
let mut decoder = gif::Decoder::new(File::open("tests/samples/sample_1.gif").unwrap());
// Configure the decoder such that it will expand the image to RGBA.
decoder.set(gif::ColorOutput::RGBA);
// Read the file header
let mut decoder = decoder.read_info().unwrap();
while let Some(frame) = decoder.read_next_frame().unwrap() {
    // Process every frame
}
```

### Encoding GIF files

The encoder can be used so save simple computer generated images:

```
use gif::{Frame, Encoder};
use std::fs::File;
use std::borrow::Cow;

let color_map = &[0, 0, 0, 0xFF, 0xFF, 0xFF];
let mut frame = Frame::default();
let mut buffer = Vec::new();
// Generate checkerboard lattice
for (i, j) in (0..10).zip(0..10) {
	buffer.push(if (i * j) % 2 == 0 {
		1
	} else {
		0
	})
}
frame.buffer = Cow::Owned(buffer);
let mut image = Vec::new();
let mut encoder = Encoder::new(&mut image, 100, 100);
encoder.write_global_palette(color_map).unwrap().write_frame(&frame).unwrap();
```

`Frame::from_*` can be used to convert a true color image to a paletted
image with a maximum of 256 colors:

```
use std::fs::File;

// Get pixel data from some source
let mut pixels: Vec<u8> = vec![0; 30_000];
// Create frame from data
let frame = gif::Frame::from_rgb(100, 100, &mut *pixels);
// Create encoder
let mut image = Vec::new();
let encoder = gif::Encoder::new(&mut image, frame.width, frame.height);
// Write header to file
let mut encoder = encoder.write_global_palette(&[]).unwrap();
// Write frame to file
encoder.write_frame(&frame).unwrap();
```
