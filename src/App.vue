<template>
  <v-app>
    <v-navigation-drawer permanent absolute app :width="280">
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title class="title">
            Single-Line Font Renderer
          </v-list-item-title>
          <!-- <v-list-item-subtitle> subtext </v-list-item-subtitle> -->
        </v-list-item-content>
      </v-list-item>

      <v-divider></v-divider>

      <v-list>
        <v-list-item>
          <v-list-item-content>
            <v-textarea
              label="Input"
              v-model="text"
              @input="render"
              outlined
            ></v-textarea>

            <v-container class="font pa-0 mb-8">
              <v-subheader class="mb-2">Font</v-subheader>

              <v-row class="mx-0 my-2">
               {{ selectedFontName }}
              </v-row>
              
              <v-row class="mx-0 my-4">
                <v-btn-toggle
                  v-model="toggle_exclusive"
                  mandatory
                  small
                >
                  <v-btn small @click="useCustomFont = false">
                    <v-icon small >mdi-format-list-bulleted</v-icon> 
                  </v-btn>
                  <v-btn small @click="useCustomFont = true">
                    <v-icon small >mdi-paperclip</v-icon>
                  </v-btn>
                </v-btn-toggle>
              </v-row>

              <template v-if="!useCustomFont">
                <v-subheader class="mb-2">Default fonts</v-subheader>
                <v-select
                  class="mt-1"
                  :items="fonts"
                  item-text="displayName"
                  dense
                  v-model="selectedFontName"
                  @input="selectFont"
                ></v-select>
              </template>
              <template v-else>
                <v-subheader class="mb-2">Load custom SVG font file</v-subheader>
                <v-file-input
                  class="mt-1 pt-0"
                  accept=".svg"
                  dense
                  single-line
                  v-model="customFontInput"
                  label="Select file …"
                  @change="loadLocalFont"
                ></v-file-input>
              </template>
            </v-container>
            <div>
              <v-subheader>Scale</v-subheader>
              <v-slider
                @input="applyFontScale"
                v-model="fontScale"
                step="0.01"
                max="10"
                min="0.1"
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
                @input="applyStrokeWeight"
                v-model="strokeWeight"
                step="0.01"
                max="20"
                min="0.1"
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
            <div>
              <v-checkbox
                @change="applyFontSettings(true)"
                v-model="enableFontSimplification"
                label="Simplify (experimental)"
                dense
                class="ma-0"
              ></v-checkbox>
              <div v-if="enableFontSimplification">
                <v-subheader>Simplify Deviation</v-subheader>
                <v-slider
                  @input="applyFontSettings(true)"
                  v-model="simplifyFactor"
                  step="1"
                  max="30"
                  min="0"
                >
                  <template v-slot:append>
                    <v-text-field
                      v-model="simplifyFactor"
                      class="mt-0 pt-0"
                      type="number"
                      style="width: 45px"
                      dense
                    ></v-text-field>
                  </template>
                </v-slider>
              </div>
            </div>
            <div>
              <v-checkbox
                @change="applyFontSettings(true)"
                v-model="enableFontSmoothing"
                label="Smooth (experimental)"
                dense
                class="ma-0"
              ></v-checkbox>

              <v-select
                v-if="enableFontSmoothing"
                @input="applyFontSettings(true)"
                :items="smoothingTypes"
                label="Smooth Type"
                v-model="smoothingType"
              ></v-select>

              <div v-if="enableFontSmoothing && smoothingType == 'catmull-rom'">
                <v-subheader>Smooth amount</v-subheader>
                <v-slider
                  @input="applyFontSettings(true)"
                  v-model="smoothingFactor"
                  step="0.01"
                  max="1"
                  min="0.01"
                >
                  <template v-slot:append>
                    <v-text-field
                      v-model="smoothingFactor"
                      class="mt-0 pt-0"
                      type="number"
                      style="width: 45px"
                      dense
                    ></v-text-field>
                  </template>
                </v-slider>
              </div>
            </div>

            <v-btn color="primary" block elevation="2" @click="exportSVG"
              >Download SVG</v-btn
            >
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
          >Improve these fonts<v-icon right> mdi-pencil </v-icon></v-btn
        >
        <v-btn
          large
          plain
          href="https://github.com/jvolker/single-line-font-renderer"
          target="_blank"
          >Improve this tool<v-icon right> mdi-pencil </v-icon></v-btn
        >
      </template>
    </v-navigation-drawer>
    <v-main>
      <v-container id="alerts">
        <v-row class="mt-2" align="center" justify="center">
          <v-alert
            v-model="error"
            dismissible
            type="error"
          >Couldn't load local font file.</v-alert>
        </v-row>
      </v-container>
      <v-container fill-height fluid id="svgWrapper">
        <v-row align="center" justify="center">
          <v-col>
            <svg
              ref="svg"
              xmlns="http://www.w3.org/2000/svg"
              xmlns:xlink="http://www.w3.org/1999/xlink"
              version="1.1"
              v-html="displayedSvgContent"
              :width="svgWidth"
              :height="svgHeight"
            ></svg>
          </v-col> 
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import _axios from "axios";
import { setupCache } from "axios-cache-adapter";
import svgFontRenderer from "./lib/svgFontRenderer";
import paper from "paper";

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

const defaults = {
  selectedFontName : "HersheySans1"
}

export default {
  name: "App",

  data() {
    return {
      fonts: [],
      selectedFontName: null,
      useCustomFont: false,
      customFontInput: null,
      strokeWeight: 1,
      fontScale: 1,
      enableFontSimplification: false,
      simplifyFactor: 0,
      enableFontSmoothing: false,
      smoothingTypes: ["catmull-rom", "geometric", "continuous", "asymmetric"],
      smoothingType: "catmull-rom",
      smoothingFactor: 0.5,
      right: null,
      rawSvgContent: null,
      displayedSvgContent: null,
      svgWidth: null,
      svgHeight: null,
      rawSVGWidth: null,
      rawSVGHeight: null,
      text:
        "The Woodman set to work at once, and so sharp was his axe that the tree was soon chopped nearly through.",
      renderedSVG: null,
      error: null
    };
  },
  mounted: async function () {
    await this.getFonts();
    this.reset();
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
          const receivedFonts = response.data
            .map((file) => {
              return {
                fileName: file.name,
                displayName: file.name.replace(".svg", ""),
                basePath: basePath,
              };
            })
            .filter((font) => font.fileName.endsWith(".svg"));
          component.fonts = [...component.fonts, ...receivedFonts];
        });
      }
    },
    async selectFont() {
      const path = `${this.selectedFont.basePath}/${this.selectedFont.fileName}`;
      const url = `https://gitlab.com/api/v4/projects/12941474/repository/files/${encodeURIComponent(
        path
      )}/raw?ref=master`;
      await axios
        .get(url)
        .then((response) => {
          const fontName = svgFontRenderer.addSVGFontFromData(response.data);
          this.$set(this.selectedFont, "fontName", fontName);
        })
        .then(this.render);
    },
    async loadLocalFont(file) {
      if (!file) { 
        // reset to default font after ejecting local font
        this.reset();
        return;
      } 

      const reader = new FileReader();
      reader.readAsText(file);

      reader.onload = () => {
        const fontName = svgFontRenderer.addSVGFontFromData(reader.result);
        if (fontName){
          this.error = false;
          this.$set(this.selectedFont, "fontName", fontName);
          this.render();
        } else {
          this.error = true;
          this.customFontInput = null;
          this.reset();
        }

      };
    },
    render() {
      this.displayedSvgContent = this.rawSvgContent = svgFontRenderer.renderTextSVG(
        this.text,
        {
          font: this.selectedFont.fontName,
          scale: 0.1,
        }
      );

      // wait for next tick when svg has rendered
      this.$nextTick(() => {
        this.getSVGSize(true);
        this.applyFontSettings();
      });
    },
    applyFontSettings(triggerSmoothing = true) {
      const applyDomSettings = () => {
        this.applyStrokeWeight();
        this.applyFontScale();
      };
      if (triggerSmoothing) this.smoothFont(applyDomSettings);
      else applyDomSettings();
    },
    applyFontScale() {
      if (!this.$refs.svg) return;

      const fontWrapper = this.$refs.svg.firstChild;
      fontWrapper.setAttribute("transform", `scale(${this.fontScale})`);

      this.getSVGSize();
    },
    applyStrokeWeight() {
      if (!this.$refs.svg) return;
      const fontWrapper = this.$refs.svg.firstChild;
      fontWrapper.setAttribute("stroke-width", this.strokeWeight);
    },
    getSVGSize(raw = false) {
      const bbox = this.$refs.svg.getBBox();

      // Update the width and height using the size of the contents
      this.svgWidth = Math.ceil(bbox.x + bbox.width + bbox.x);
      this.svgHeight = Math.ceil(bbox.y + bbox.height + bbox.y);
      if (raw) {
        this.rawSVGWidth = this.svgWidth;
        this.rawSVGHeight = this.svgHeight;
      }
    },
    smoothFont(callback) {
      if (
        !this.rawSVGWidth ||
        !this.rawSVGHeight ||
        this.rawSVGWidth == 0 ||
        this.rawSVGHeight == 0
      )
        return;

      // feeding the svg into another engine might be slow especially with large text ??
      paper.setup([this.rawSVGWidth, this.rawSVGHeight]);

      const header = `<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" >`;
      const footer = "</svg>";

      paper.project.importSVG(
        `${header}\n${this.rawSvgContent}\n${footer}`,
        (item) => {
          if (!this.enableFontSimplification && !this.enableFontSmoothing)
            return;

          const letterPaths = item.children[0].children[0].children;

          letterPaths.forEach((path) => {
            if (this.enableFontSimplification)
              path.simplify(this.simplifyFactor);
            if (this.enableFontSmoothing)
              path.smooth({
                type: this.smoothingType,
                factor: this.smoothingFactor,
              });
          });
        }
      );

      this.displayedSvgContent = paper.project.exportSVG().innerHTML;

      this.$nextTick(() => {
        if (callback) callback();
      });
    },
    exportSVG() {
      console.log("export svg");
      const string = this.$refs.svg.outerHTML; // raw svg
      if (!string) return;
      const blob = new Blob([string], { type: "image/svg+xml;charset=utf-8" });
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download =
        truncate(this.text.replace(/[^a-z0-9]/gi, "-").toLowerCase(), 60) +
        ".svg"; // download file name
      a.click();
      window.URL.revokeObjectURL(url);
    },
    async reset() {    
      this.selectedFontName = defaults.selectedFontName;
      await this.selectFont();
      this.render();
    },
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

#svgWrapper {
  overflow: scroll;
}

#alerts {
  position: absolute;
}

.font .v-text-field__details {
  display: none;
}
</style>