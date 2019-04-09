# PDF Reader

A simple Go library which enables reading PDF files. Forked from https://github.com/rsc/pdf

Features
  - Get plain text content (without format)
  - Get Content (including all font and formatting information)

## Install:

`go get -u github.com/ledongthuc/pdf`


## Read plain text

```golang
package main

import (
	"bytes"
	"fmt"

	"github.com/ledongthuc/pdf"
)

func main() {
	content, err := readPdf("test.pdf") // Read local pdf file
	if err != nil {
		panic(err)
	}
	fmt.Println(content)
	return
}

func readPdf(path string) (string, error) {
	// Open the file f, as reader r, and put errors in err.
	f, r, err := pdf.Open(fileName)
	if err != nil {
		return "", err
	}
	defer f.Close()

	// Extract number of pages.
	totalPage := r.NumPage()

	var textBuilder bytes.Buffer
	for pageIndex := 1; pageIndex <= totalPage; pageIndex++ {
		page := r.Page(pageIndex)
		if page.V.IsNull() {
			continue
		}
		// Extract font for special characters and accented letters.
		fonts := make(map[string]*pdf.Font)
		pageFonts := page.Fonts()
		for _, n := range pageFonts {
			pointer := page.Font(n)
			fonts[n] = &pointer
		}

		text, _ := page.GetPlainText(fonts)
		textBuilder.WriteString(text)
	}

	return textBuilder.String(), nil
}
```

## Read all text with styles from PDF

```golang
func readPdf2(path string) (string, error) {
	f, r, err := pdf.Open(path)
	// remember close file
	defer f.Close()
	if err != nil {
		return "", err
	}
	totalPage := r.NumPage()

	for pageIndex := 1; pageIndex <= totalPage; pageIndex++ {
		p := r.Page(pageIndex)
		if p.V.IsNull() {
			continue
		}
		var lastTextStyle pdf.Text
		texts := p.Content().Text
		for _, text := range texts {
			if isSameSentence(text, lastTextStyle) {
				lastTextStyle.S = lastTextStyle.S + text.S
			} else {
				fmt.Printf("Font: %s, Font-size: %f, x: %f, y: %f, content: %s \n", lastTextStyle.Font, lastTextStyle.FontSize, lastTextStyle.X, lastTextStyle.Y, lastTextStyle.S)
				lastTextStyle = text
			}
		}
	}
	return "", nil
}
```

## Demo
![Run example](https://i.gyazo.com/01fbc539e9872593e0ff6bac7e954e6d.gif)
