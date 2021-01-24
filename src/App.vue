<template>
  <v-app>
    <v-navigation-drawer permanent absolute app :width="280">
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title class="title">
            SVG Font Renderer
          </v-list-item-title>
          <v-list-item-subtitle> subtext </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>

      <v-divider></v-divider>

      <v-list>
        <v-list-item>
          <v-list-item-content>
            <v-textarea
              name="input-7-1"
              label="Input text"
              v-model="text"
              @input="render"
              outlined
            ></v-textarea>
            <v-select
              :items="fonts"
              item-text="displayName"
              label="Font"
              dense
              v-model="selectedFontName"
              @input="selectFont"
            ></v-select>
            <div>
              <v-subheader>Font Scale</v-subheader>
              <v-slider
                @input="setFontScale"
                v-model="fontScale"
                step="0.01"
                max="1"
                min="0.01"
                block
              >
                <template v-slot:append>
                  <v-text-field
                    v-model="fontScale"
                    class="mt-0 pt-0"
                    type="number"
                    step="0.01"
                    style="width: 45px"
                    dense
                  ></v-text-field>
                </template>
              </v-slider>
            </div>
            <div>
              <v-subheader>Stroke width</v-subheader>
              <v-slider
                @input="setStrokeWeight"
                v-model="strokeWeight"
                step="0.1"
                max="50"
                min="0.05"
              >
                <template v-slot:append>
                  <v-text-field
                    v-model="strokeWeight"
                    class="mt-0 pt-0"
                    type="number"
                    style="width: 45px"
                    dense
                  ></v-text-field>
                </template>
              </v-slider>
            </div>
            <v-btn block elevation="2" @click="exportSVG">Download SVG</v-btn>
          </v-list-item-content>
        </v-list-item>
      </v-list>
      <template v-slot:append>
        <v-divider></v-divider>
        <v-btn
          large
          plain
          href="https://gitlab.com/oskay/svg-fonts"
          target="_blank"
          ><v-icon left> mdi-pencil </v-icon>Improve these fonts</v-btn
        >
        <v-btn
          large
          plain
          href="https://github.com/jvolker/svg-font-renderer"
          target="_blank"
          ><v-icon left> mdi-pencil </v-icon>Improve this tool</v-btn
        >
      </template>
    </v-navigation-drawer>
    <v-content
      ><v-container fill-height fluid>
        <v-row align="center" justify="center">
          <v-col>
            <div ref="svgWrapper" v-html="renderedSVG"></div>
          </v-col> </v-row
      ></v-container>
    </v-content>
  </v-app>
</template>

<script>
import _axios from "axios";
import { setupCache } from "axios-cache-adapter";
import svgFontRenderer from "./lib/svgFontRenderer";

// Create `axios-cache-adapter` instance
const cache = setupCache({
  maxAge: 24 * 60 * 60 * 1000, // cache for 24h
});

// Create `axios` instance passing the newly created `cache.adapter`
const axios = _axios.create({
  adapter: cache.adapter,
});

function truncate(str, n) {
  return str.length > n ? str.substr(0, n - 1) : str;
}

export default {
  name: "App",

  data() {
    return {
      fonts: [],
      selectedFontName: null,
      strokeWeight: 20,
      fontScale: 0.1,
      right: null,
      text:
        "The Woodman set to work at once, and so sharp was his axe that the tree was soon chopped nearly through.",
      renderedSVG: null,
    };
  },
  mounted: async function () {
    await this.getFonts();
    this.selectedFontName = "HersheySans1";
    await this.selectFont();
    this.render();
  },
  computed: {
    selectedFont: function () {
      return this.fonts.find(
        (font) => font.displayName === this.selectedFontName
      );
    },
  },
  methods: {
    async getFonts() {
      const component = this;

      await getFontList("fonts/EMS");
      await getFontList("fonts/Hershey");

      async function getFontList(basePath) {
        const url = `https://gitlab.com/api/v4/projects/12941474/repository/tree?path=${basePath}&per_page=100`;
        await axios.get(url).then((response) => {
            const receivedFonts = response.data.map((font) => {
            return {
              fileName: font.name,
              displayName: font.name.replace(".svg", ""),
              basePath: basePath,
            };
          });
          component.fonts = [...component.fonts, ...receivedFonts];
        });
      }
    },
    async selectFont() {
      // const selectedFont = this.fonts.find(font => font.displayName === this.selectedFontName);
      const path = `${this.selectedFont.basePath}/${this.selectedFont.fileName}`;
      const url = `https://gitlab.com/api/v4/projects/12941474/repository/files/${encodeURIComponent(
        path
      )}/raw?ref=master`;
      await axios
        .get(url)
        .then((response) => {
          const fontName = svgFontRenderer.addSVGFontFromData(response.data);
          // this.$set(selectedFont, 'fontData',response.data);
          this.$set(this.selectedFont, "fontName", fontName);
        })
        .then(this.render);
    },
    render() {
      const svgContent = svgFontRenderer.renderTextSVG(this.text, {
        font: this.selectedFont.fontName,
        scale: 0.1,
      });

      const header =
        '<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" >';
      const footer = "</svg>";
      this.renderedSVG = `${header}\n${svgContent}\n${footer}`;

      this.$nextTick(() => {
        // wait for next tick when svg has rendered
        this.setStrokeWeight();
        this.setFontScale();
        this.resizeSVG();
      });
    },
    getSvgElement: function () {
      return this.$refs.svgWrapper.firstChild;
    },
    setFontScale() {
      if (!this.getSvgElement()) return;
      const pathWrapper = this.getSvgElement().getElementById("text");
      pathWrapper.setAttribute("transform", `scale(${this.fontScale})`);

      this.resizeSVG();
    },
    setStrokeWeight() {
      const paths = this.getSvgElement().getElementsByTagName("path");
      paths.forEach((path) =>
        path.setAttribute("stroke-width", this.strokeWeight)
      );
    },
    resizeSVG() {
      // Get the bounds of the SVG content
      const svgElement = this.getSvgElement();
      const bbox = svgElement.getBBox();

      // Update the width and height using the size of the contents
      const svgWidth = bbox.x + bbox.width + bbox.x;
      const svgHeight = bbox.y + bbox.height + bbox.y;

      svgElement.setAttribute("width", svgWidth);
      svgElement.setAttribute("height", svgHeight);

      // console.log(this.svgElement)
    },
    exportSVG() {
      console.log("export svg");
      const string = this.$refs.svgWrapper.firstChild.outerHTML; // raw svg
      if (!string) return;
      const blob = new Blob([string], { type: "image/svg+xml;charset=utf-8" });
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = truncate(
        this.text.replace(/[^a-z0-9]/gi, "_").toLowerCase(),
        100
      ); // download file name
      a.click();
      window.URL.revokeObjectURL(url);
    },
    reset() {},
  },
};
</script>


<style>
div.v-subheader {
  padding: 0;
  height: 8px;
  font-size: 12px;
}

div.v-slider--horizontal {
  margin-left: 0;
}
</style>