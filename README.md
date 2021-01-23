# svg-font-renderer

## Text renderer for CNC machines

This web based tool renders and exports text in SVG format intended for CNC machines like pen plotters or laser engravers.

These machines require a different single line font type than the more commonly used outline fonts (eg. TTF/OTF) which are mainly intended for screens and printing. More on this topic in this [article](https://www.evilmadscientist.com/2011/hershey-text-an-inkscape-extension-for-engraving-fonts/).

This tool is an attempt to create a browser based alternative to the excellent [Hershey Text Extension for Inkscape](https://wiki.evilmadscientist.com/Hershey_Text) by Evil Mad Scientist. 

[**CHECK IT OUT HERE**](https://jvolker.github.io/svg-font-renderer/)

**Advantages** of using this in a web browser over using Inkscape:
- simplified user interface
- doesn't require any installation or extension to be installed
- always uses the latest version of SVG fonts available

**Disadvantages:**
- the feature set of this tool is currently more limited

## Export and usage

The tool exports SVGs to be used and rearranged in a vector/graphic design software like Adobe Illustrator. 

I personally prefer [Affinity Designer](https://affinity.serif.com/en-gb/designer/) and then plot the results on an [Axidraw pen plotter](https://axidraw.com/) using [saxi](https://github.com/nornagon/saxi/), a web based control software for Axidraw plotters.


## Fonts

This tool makes use of SVG fonts. This format has advantages over the original Hershey font format as explained in this [article](https://www.evilmadscientist.com/2019/hershey-text-v30/). The fonts are sourced from a [repositiory with SVG fonts](https://gitlab.com/oskay/svg-fonts) maintained by Evil Mad Scientist. It contains updated Hershey and other fonts converted and shared by a variety of people. Lots of them include more glyphs and therefore better language support than the original fonts.

This tool uses the latest fonts straight from that repository. Please contribute to that repository to improve the variety of fonts and their language support.

**Font licenses:** Before using any of the fonts, please make sure to check their license contained in that repository.

## Acknowledgement

The rendering core is heavily borrowed from: https://github.com/techninja/hersheytextjs  
Thanks to [Evil Mad Scientist](https://www.evilmadscientist.com/) for their pioneering work in this field.

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
