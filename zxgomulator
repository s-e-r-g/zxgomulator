package main

import (
	"image/color"
	"math/rand"
	"time"

	"fyne.io/fyne/v2"
	"fyne.io/fyne/v2/app"
	"fyne.io/fyne/v2/canvas"
	"fyne.io/fyne/v2/widget"
)

const (
	topScreenOffset  = 60
	leftScreenOffset = 40
	screenWidth      = 256
	screenHeight     = 192
)

var memory [65536]uint8

func mem_read(address uint16) uint8 {
	return memory[address]
}

func mem_write(address uint16, value uint8) {
	memory[address] = value
}

func getAttrColor(code uint8, brightnessBit uint8) color.RGBA {
	b := code & 1
	r := (code >> 1) & 1
	g := (code >> 2) & 1
	r <<= (6 + brightnessBit)
	g <<= (6 + brightnessBit)
	b <<= (6 + brightnessBit)
	return color.RGBA{r, g, b, 0xff}
}

func getAttrInkColor(attr uint8) color.RGBA {
	return getAttrColor(attr&7, (attr>>6)&1)
}

func getAttrPaperColor(attr uint8) color.RGBA {
	return getAttrColor((attr>>3)&7, (attr>>6)&1)
}

func zxRaster(x, y, w, h int) color.Color {
	scale := 3
	x /= scale
	y /= scale

	// check if it's border
	if y < topScreenOffset || x < leftScreenOffset || x >= (leftScreenOffset+screenWidth) || y >= (topScreenOffset+screenHeight) {
		borderColor := color.RGBA{0, 0, 0, 0xff}
		return borderColor
	}

	// truncate x to [0..255] and y to [0..191]
	x -= leftScreenOffset
	y -= topScreenOffset
	//

	bitNumber := uint8(x % 8)

	// TODO: make proper formula here
	portion := y >> 6 // 0..2
	rest := y & 63

	line := (rest&7)<<3 + (rest >> 3)
	address := uint16(16384 + x>>3 + line<<5 + portion<<11)

	bit := mem_read(address) & bitNumber

	attrAddress := uint16(22528 + x>>3 + (y>>3)<<5)

	attr := mem_read(attrAddress)

	if bit == 0 {
		return getAttrPaperColor(attr)
	}

	return getAttrInkColor(attr)
}

func main() {
	a := app.New()
	w := a.NewWindow("Hello World")
	w.Resize(fyne.NewSize(640, 480))

	w.SetContent(widget.NewLabel("Hello World!"))
	raster := canvas.NewRasterWithPixels(zxRaster)

	w.SetContent(raster)

	go func() {
		for i := 22528; i < 65536; i++ {
			mem_write(uint16(i), 7)
		}
		for i := 16384; i < 65536; i++ {
			mem_write(uint16(i), uint8(rand.Intn(256)))
			time.Sleep(10 * time.Millisecond)
			raster.Refresh()
		}
	}()

	w.ShowAndRun()
}
