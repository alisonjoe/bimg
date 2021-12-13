# bimg [![Build Status](https://travis-ci.org/h2non/bimg.svg)](https://travis-ci.org/h2non/bimg) [![GoDoc](https://godoc.org/github.com/h2non/bimg?status.svg)](https://godoc.org/github.com/h2non/bimg) [![Go Report Card](http://goreportcard.com/badge/h2non/bimg)](http://goreportcard.com/report/h2non/bimg) [![Coverage Status](https://coveralls.io/repos/github/h2non/bimg/badge.svg?branch=master)](https://coveralls.io/github/h2non/bimg?branch=master) ![License](https://img.shields.io/badge/license-MIT-blue.svg)

Small [Go](http://golang.org) package for fast high-level image processing using [libvips](https://github.com/jcupitt/libvips) via C bindings, providing a simple [programmatic API](#examples).

bimg was designed to be a small and efficient library supporting common [image operations](#supported-image-operations) such as crop, resize, rotate, zoom or watermark. It can read JPEG, PNG, WEBP natively, and optionally TIFF, PDF, GIF and SVG formats if `libvips@8.3+` is compiled with proper library bindings. Lastly AVIF is supported as of `libvips@8.9+`. For AVIF support `libheif` needs to be [compiled with an applicable AVIF en-/decoder](https://github.com/strukturag/libheif#compiling).

bimg is able to output images as JPEG, PNG and WEBP formats, including transparent conversion across them.

bimg uses internally libvips, a powerful library written in C for image processing which requires a [low memory footprint](https://github.com/jcupitt/libvips/wiki/Speed_and_Memory_Use)
and it's typically 4x faster than using the quickest ImageMagick and GraphicsMagick settings or Go native `image` package, and in some cases it's even 8x faster processing JPEG images.

If you're looking for an HTTP based image processing solution, see [imaginary](https://github.com/h2non/imaginary).

bimg was heavily inspired in [sharp](https://github.com/lovell/sharp), its homologous package built for [node.js](http://nodejs.org). bimg is used in production environments processing thousands of images per day.

**v1 notice**: `bimg` introduces some minor breaking changes in `v1` release.
If you're using `gopkg.in`, you can still rely in the `v0` without worrying about API breaking changes.

## Contents

- [Supported image operations](#supported-image-operations)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Performance](#performance)
- [Benchmark](#benchmark)
- [Examples](#examples)
- [Debugging](#debugging)
- [API](#api)
- [Authors](#authors)
- [Credits](#credits)

## Supported image operations

- Resize
- Enlarge
- Crop (including smart crop support, libvips 8.5+)
- Rotate (with auto-rotate based on EXIF orientation)
- Flip (with auto-flip based on EXIF metadata)
- Flop
- Zoom
- Thumbnail
- Extract area
- Watermark (using text or image)
- Gaussian blur effect
- Custom output color space (RGB, grayscale...)
- Format conversion (with additional quality/compression settings)
- EXIF metadata (size, alpha channel, profile, orientation...)
- Trim (libvips 8.6+)

## Prerequisites

- [libvips](https://github.com/libvips/libvips) 8.3+ (8.10+ recommended)
- C compatible compiler such as gcc 4.6+ or clang 3.0+
- Go 1.17+

**Note**: 
 * `libvips` v8.3+ is required for GIF, PDF and SVG support.
 * `libvips` v8.9+ is required for AVIF support. `libheif` compiled with a AVIF en-/decoder also needs to be present.
 * `libvips` v8.11+ is required for JPEG2000 support.

## Installation

```bash
go get -u github.com/h2non/bimg/v2
```

### libvips

Follow `libvips` installation instructions:

[https://libvips.github.io/libvips/install.html](https://libvips.github.io/libvips/install.html)

## Performance

libvips is probably the fastest open source solution for image processing.
Here you can see some performance test comparisons for multiple scenarios:

- [libvips speed and memory usage](https://github.com/jcupitt/libvips/wiki/Speed-and-memory-use)

## Benchmark

Tested using Go 1.5.1 and libvips-7.42.3 in OSX i7 2.7Ghz
```
BenchmarkRotateJpeg-8     	      20	  64686945 ns/op
BenchmarkResizeLargeJpeg-8	      20	  63390416 ns/op
BenchmarkResizePng-8      	     100	  18147294 ns/op
BenchmarkResizeWebP-8     	     100	  20836741 ns/op
BenchmarkConvertToJpeg-8  	     100	  12831812 ns/op
BenchmarkConvertToPng-8   	      10	 128901422 ns/op
BenchmarkConvertToWebp-8  	      10	 204027990 ns/op
BenchmarkCropJpeg-8       	      30	  59068572 ns/op
BenchmarkCropPng-8        	      10	 117303259 ns/op
BenchmarkCropWebP-8       	      10	 107060659 ns/op
BenchmarkExtractJpeg-8    	      50	  30708919 ns/op
BenchmarkExtractPng-8     	    3000	    595546 ns/op
BenchmarkExtractWebp-8    	    3000	    386379 ns/op
BenchmarkZoomJpeg-8       	      10	 160005424 ns/op
BenchmarkZoomPng-8        	      30	  44561047 ns/op
BenchmarkZoomWebp-8       	      10	 126732678 ns/op
BenchmarkWatermarkJpeg-8  	      20	  79006133 ns/op
BenchmarkWatermarPng-8    	     200	   8197291 ns/op
BenchmarkWatermarWebp-8   	      30	  49360369 ns/op
```

## Examples

```go
import (
  "fmt"
  "os"
  "github.com/h2non/bimg/v2"
)
```

#### Resize

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).Resize(800, 600)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

size, err := bimg.NewImage(newImage).Size()
if size.Width == 800 && size.Height == 600 {
  fmt.Println("The image size is valid")
}

bimg.Write("new.jpg", newImage)
```

#### Rotate

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).Rotate(90)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

bimg.Write("new.jpg", newImage)
```

#### Convert

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).Convert(bimg.PNG)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

if bimg.NewImage(newImage).Type() == "png" {
  fmt.Fprintln(os.Stderr, "The image was converted into png")
}
```

#### Force resize

Force resize operation without perserving the aspect ratio:

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).ForceResize(1000, 500)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

size := bimg.Size(newImage)
if size.Width != 1000 || size.Height != 500 {
  fmt.Fprintln(os.Stderr, "Incorrect image size")
}
```

#### Custom colour space (black & white)

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).Colourspace(bimg.INTERPRETATION_B_W)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

colourSpace, _ := bimg.ImageInterpretation(newImage)
if colourSpace != bimg.INTERPRETATION_B_W {
  fmt.Fprintln(os.Stderr, "Invalid colour space")
}
```

#### Custom options

See [Options](https://godoc.org/github.com/h2non/bimg#Options) struct to discover all the available fields

```go
options := bimg.Options{
  Width:        800,
  Height:       600,
  Crop:         true,
  Quality:      95,
  Rotate:       180,
  Interlace:    true,
}

buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

newImage, err := bimg.NewImage(buffer).Process(options)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

bimg.Write("new.jpg", newImage)
```

#### Watermark

```go
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

watermark := bimg.Watermark{
  Text:       "Chuck Norris (c) 2315",
  Opacity:    0.25,
  Width:      200,
  DPI:        100,
  Margin:     150,
  Font:       "sans bold 12",
  Background: bimg.Color{255, 255, 255},
}

newImage, err := bimg.NewImage(buffer).Watermark(watermark)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
}

bimg.Write("new.jpg", newImage)
```

#### Fluent interface

If you intend to apply multiple transformations to the same image, you
should use the `ImageTransformation` system which will decode the image
once and lets you work on this decoded image as long as you please.

```go
// read the raw data
buffer, err := bimg.Read("image.jpg")
if err != nil {
  fmt.Fprintln(os.Stderr, err)
  return
}

// decode the image and prepare our transformation chain
it, err := bimg.NewImageTransformation(buffer)
if err != nil {
  fmt.Fprintln(os.Stderr, err)
  return
}

// crop the image to a specific width
err = it.Crop(bimg.CropOptions{Width: 300})
if err != nil {
  fmt.Fprintln(os.Stderr, err)
  return
}

// then flip it
err = it.Rotate(bimg.RotateOptions{Flip: true})
if err != nil {
  fmt.Fprintln(os.Stderr, err)
  return
}

// encode the image
newImage, err := it.Save(bimg.SaveOptions{})
if err != nil {
  fmt.Fprintln(os.Stderr, err)
  return
} 

// save the cropped and flipped image
bimg.Write("new.jpg", newImage)
```

## Debugging

Run the process passing the `DEBUG` environment variable
```
DEBUG=bimg ./app
```

Enable libvips traces (note that a lot of data will be written in stdout):
```
VIPS_TRACE=1 ./app
```

You can also dump a core on failure, as [John Cuppit](https://github.com/jcupitt) said:
```c
g_log_set_always_fatal(
                G_LOG_FLAG_RECURSION |
                G_LOG_FLAG_FATAL |
                G_LOG_LEVEL_ERROR |
                G_LOG_LEVEL_CRITICAL |
                G_LOG_LEVEL_WARNING );
```

Or set the G_DEBUG environment variable:
```
export G_DEBUG=fatal-warnings,fatal-criticals
```

## API

See [godoc reference](https://godoc.org/github.com/h2non/bimg) for detailed API documentation.

## Authors

- [Tomás Aparicio](https://github.com/h2non) - Original author and architect.

## Credits

People who recurrently contributed to improve `bimg` in some way.

- [John Cupitt](https://github.com/jcupitt)
- [Yoan Blanc](https://github.com/greut)
- [Christophe Eblé](https://github.com/chreble)
- [Brant Fitzsimmons](https://github.com/bfitzsimmons)
- [Thomas Meson](https://github.com/zllak)

Thank you!

## License

MIT - Tomas Aparicio

[![views](https://sourcegraph.com/api/repos/github.com/h2non/bimg/.counters/views.svg)](https://sourcegraph.com/github.com/h2non/bimg)
