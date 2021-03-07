# Single-Line Font Renderer

## Text renderer for CNC machines

This web based tool renders and exports text (currently from SVG format) intended for CNC machines like pen plotters or laser engravers.

These machines require a different single-line font type than the more commonly used outline fonts (eg. TTF/OTF) which are mainly intended for screens and printing. More on this topic in this [article](https://www.evilmadscientist.com/2011/hershey-text-an-inkscape-extension-for-engraving-fonts/).

This tool is an attempt to create a browser based alternative to the excellent [Hershey Text Extension for Inkscape](https://wiki.evilmadscientist.com/Hershey_Text) by Evil Mad Scientist. It has a simplified interface and doesn't require installation. On the downside, this tools feature set is currently more limited.

[**>> CHECK IT OUT HERE <<**](https://jvolker.github.io/single-line-font-renderer/)

<img width="1136" alt="Screenshot of the software" src="https://user-images.githubusercontent.com/546852/105638363-85436000-5e72-11eb-801a-60d2b2ce9a65.png">

**Features:**
- uses latest fonts from the [SVG fonts repository](https://gitlab.com/oskay/svg-fonts)
- adjust font scale and stroke width 
- smoothing/simplification (this is experimental: the results from Inkscape are better in some cases)
- export as SVG file

**Ideas for furture features:**
- [ ] line breaks
- [ ] Use meaningful units: eg. font size and stroke width in mm
- [ ] import local svg font files
- [ ] support for otf/ttf files (eg. using https://github.com/opentypejs/opentype.js): some single-line fonts are set in these types and hard to use in other software
- [ ] DXF export (eg. using https://github.com/microsoft/maker.js/issues/480 or https://github.com/jscad/io/tree/master/packages/svg-deserializer and https://github.com/jscad/io/tree/master/packages/dxf-serializer)
- [ ] move svg font renderer into separate repo with Node.js support and NPM release 
- [ ] text area: words wrap to new line when longer than a certain length
- [ ] alignment: right and center

## Export and usage

There are many different tools and ways to use this with CNC machines depending on the machine and use-case. Single-line font renderer was built with the following workflow in mind:

1. This tool exports SVGs to be used and rearranged in a vector/graphic design software like [Affinity Designer](https://affinity.serif.com/en-gb/designer/), Adobe Illustrator or others.
2. The results can then, for example, be used on an [Axidraw pen plotter](https://axidraw.com/) using [saxi](https://github.com/nornagon/saxi/), a web based control software for Axidraw plotters.


## Fonts

This tool makes use of SVG fonts. This format has advantages over the original Hershey font format as explained in this [article](https://www.evilmadscientist.com/2019/hershey-text-v30/). The fonts are sourced from a [repositiory with SVG fonts](https://gitlab.com/oskay/svg-fonts) maintained by Evil Mad Scientist. It contains updated Hershey and other fonts converted and shared by a variety of people. Lots of them include more glyphs and therefore better language support than the original fonts.

This tool uses the latest fonts straight from that repository. Please contribute to that repository to improve the variety of fonts and their language support.

**Font licenses:** Before using any of the fonts, please make sure to check their license contained in that repository.

## Acknowledgement

The rendering core is heavily borrowed from: https://github.com/techninja/hersheytextjs  
Thanks to [Evil Mad Scientist](https://www.evilmadscientist.com/) for their pioneering work in this field.

## Alternatives to this tool
- [Hershey Text Extension for Inkscape](https://wiki.evilmadscientist.com/Hershey_Text) by Evil Mad Scientist
- [CNC Text Tool](msurguy.github.io/cnc-text-tool/)

## Development

### Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
