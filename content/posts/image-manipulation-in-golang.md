+++
date = "2018-04-29T17:22:02+12:00"
title = "The Basics of Image Manipulation in Golang"
Categories = ["Golang", "Usergroup"]
menu = "main"
Description = "mmmmm"
Draft = true
+++

Motivation

Media is a source of bloat on the web
Users don't want to manage their own compression
Thumbnails

GOALS
* Create an image
* Compression
* Conversion
* Resizing
* Combining
* Stripping metadata

The source of truth
https://golang.org/pkg/image

Maybe something by Ben Johnson?
https://medium.com/@benbjohnson

Images in golang are represented by the interface `image.Image` which has the following properties:

```go
// Image is a finite rectangular grid of color.Color values taken from a color model.
type Image interface {
    // ColorModel returns the Image's color model.
    ColorModel() color.Model
    // Bounds returns the domain for which At can return non-zero color.
    // The bounds do not necessarily contain the point (0, 0).
    Bounds() Rectangle
    // At returns the color of the pixel at (x, y).
    // At(Bounds().Min.X, Bounds().Min.Y) returns the upper-left pixel of the grid.
    // At(Bounds().Max.X-1, Bounds().Max.Y-1) returns the lower-right one.
    At(x, y int) color.Color
}
```

Using the go guru implementation lookup gives

    libexec/src/image/image.go | pointer type implementations
        *Alpha
        *Alpha16
        *CMYK
        *Gray16
        *NRGBA
        *NRGBA64
        *Paletted
        *RGBA
        *RGBA64

    libexec/src/image/ycbcr.go | pointer type *NYCbCrA
    libexec/src/image/names.go | pointer type *Uniform
    libexec/src/image/ycbcr.go | pointer type *YCbCr
    libexec/src/image/geom.go  | struct type Rectangle
    libexec/src/image/draw/draw.go | interface type image/draw.Image

    --Derived from :GoImplements on the Image interface within vim-go

Using one of these implementations gives you access to tools from packages like
`image/png`, `image/jpeg` and the `image/draw` package.

Reading into the pointer type descriptions, generally a package named
`image.[type]` is just a way of managing a collection of `color.[type]`.
They're stored as unsigned 8bit 0x0 to 0xff unless you see a number to the
right of the package type.

So RGBA is an 32 bit color, and RGBA64 contains 4x uint16 values. If you think
that sounds weird, well each pixel is 4x 16 bit integers = 64 bits. That's
16 bits of red, 16 green, 16 blue and 16 alpha.

For this topic, I'm going to ignore higher order reperesentatins of colour and
go with the vanilla 32 bit variety. This means we're working with 4 x uint8
integers.

uint8 covers 0 to 255, or 0x0 to 0xff. You might recognise these values from
css, #ffffff contains 0xff in the Red, Green and Blue channels.


Lets look at the RGBA Definition:

```golang
type RGBA struct {
	// Pix holds the image's pixels, in R, G, B, A order. The pixel at
	// (x, y) starts at Pix[(y-Rect.Min.Y)*Stride + (x-Rect.Min.X)*4].
	Pix []uint8
	// Stride is the Pix stride (in bytes) between vertically adjacent pixels.
	Stride int
	// Rect is the image's bounds.
	Rect Rectangle
}
```

And those bounds will need these definitions
```golang
// Rectangle contains the points with Min.X <= X < Max.X, Min.Y <= Y < Max.Y.
type Rectangle struct {
	Min, Max Point
}

// Point is an X, Y coordinate pair. The axes increase right and down.
type Point struct {
	X, Y int
}
```

Lets make small image based on these interfaces. To do this, you need the
`image` package.

```golang
img := image.RGBA{}

// bounds min.x <= x < max.x, min.y <= y < max.y
img.Rect = image.Rectangle{
	image.Point{0, 0},
	image.Point{1, 5},
}

// Stride is the Pix stride (in bytes) between vertically adjacent pixels.
img.Stride = 4

// Pix holds the image's pixels, in R, G, B, A order
img.Pix = []uint8{
	0xff, 0x0, 0x0, 0xff, // Red
	0x0, 0xff, 0x0, 0xff, // Green
	0x0, 0x0, 0xff, 0xff, // Blue
	0x0, 0xff, 0x0, 0xff, // Green
	0xff, 0x0, 0x0, 0xff, // Red
}
```

Sweet! Now we've got an image let's compress it in Portable Network Graphics
(png) format. To do this, you need the `image/png` and `os` packages.


```golang
f, err := os.Create("test.png")
if err != nil {
	panic(err)
}
defer f.Close()

// Note, dereferencing pointer type
err = png.Encode(f, &img)
if err != nil {
	panic(err)
}
```

And voila, we've got a 5x1 pixel image going from red to blue, then back to red!

Lets revisit those goals:

GOALS

* ~~Create an image~~ DONE
* ~~Compression~~ DONE
* Conversion
* Resizing
* Combining
* Stripping metadata

To convert images we need to

1. Read the file
2. Decode the image
3. Re-encode as jpeg
4. Write out to disk

```golang
in, err := os.Open("test.png")
if err != nil {
	panic(err)
}
defer in.Close()

out, err := os.Create("test.jpg")
if err != nil {
	panic(err)
}
defer out.Close()

img, err := png.Decode(in)
if err != nil {
	panic(err)
}

opts := jpeg.Options{Quality: 100}
err = jpeg.Encode(out, img, &opts)
if err != nil {
	panic(err)
}
```

There's a few image detection libraries `github.com/h2non/filetype` 

* ~~Create an image~~ DONE
* ~~Compression~~ DONE
* ~~Conversion~~ DONE
* Resizing
* Combining
* Stripping metadata

There's a great comparison here:
https://github.com/fawick/speedtest-resize

Combining
https://blog.golang.org/go-imagedraw-package

Stripping metadata
https://gist.github.com/artyom/7640686
