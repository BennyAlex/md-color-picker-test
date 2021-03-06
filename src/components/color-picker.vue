<template>
  <div
    class="mdcp"
    :style="
      fixedMinHeight
        ? { width: wrapperWidth, minHeight: wrapperMinHeight }
        : { width: wrapperWidth }
    "
  >
    <div
      v-if="subPalette !== undefined"
      @click="subPalette = undefined"
      class="mdcp-back-icon"
      :style="{
        order: backButtonAtEnd ? 2 : 1,
        margin: colorMargin + 'px',
        height: colorSizePx,
        width: colorSizePx
      }"
    >
      <svg
        :fill="backButtonColor"
        :height="colorSize"
        :width="colorSize / 2"
        viewBox="0 0 24 24"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path d="M0 0h24v24H0z" fill="none" />
        <path
          d="M20 11H7.83l5.59-5.59L12 4l-8 8 8 8 1.41-1.41L7.83 13H20v-2z"
        />
      </svg>
    </div>

    <div
      v-for="color in colors"
      :key="color.name"
      @click.stop="click(color)"
      class="mdcp-color"
      :class="{
        'mdcp-selected':
          color.value.toLowerCase() === value.toLowerCase() ||
          isTintOfSelected(color),
        'mdcp-is-light': colorIsLight(color.value)
      }"
      :style="getColorStyle(color)"
      :title="color.name"
    >
      <span :style="circleStyle" class="mdcp-circle"></span>
    </div>
  </div>
</template>

<script>
import { colorIsLight, colorIsDark } from "../color-brightness";
import materialPalette from "../palettes/material-palette";
import accentMaterialPalette from "../palettes/material-palette-accent";
import fullMaterialPalette from "../palettes/material-palette-full";

function arrayIncludes(arr, obj) {
  let i = arr.length;
  while (i--) {
    if (arr[i] === obj) return true;
  }
  return false;
}

function valuesOfObj(obj) {
  const values = [];
  Object.keys(obj).forEach(key => {
    values.push(obj[key]);
  });
  return values;
}

export default {
  name: "color-picker",
  props: {
    value: {
      type: String,
      required: true
    },
    palette: {
      type: [String, Object],
      required: false
    },
    backButtonAtEnd: {
      type: Boolean,
      default: false
    },
    backButtonColor: {
      type: String,
      default: "#000000"
    },
    colorSize: {
      type: Number,
      default: 54
    },
    colorsPerRow: {
      type: Number,
      default: 5
    },
    colorMargin: {
      type: Number,
      default: 6
    },
    defaultTint: {
      type: [Number, String],
      default: 500
    },
    fixedMinHeight: {
      type: Boolean,
      default: true
    },
    useSpectrumPicker: {
      type: Boolean,
      default: true
    }
  },
  methods: {
    getColorStyle(color) {
      return {
        background: color.value,
        margin: this.colorMargin + "px",
        height: this.colorSizePx,
        width: this.colorSizePx
      };
    },
    colorIsLight(color) {
      return colorIsLight(color, 210);
    },
    click(color) {
      if (
        this.useSpectrumPicker &&
        typeof this.currentPalette[color.name] === "object"
      ) {
        this.subPalette = color.name;
        if (this.isTintOfSelected(color)) return;
        this.selectedColorName = color.name;
      }
      this.$emit("change", color.value);
    },
    isTintOfSelected(color) {
      return (
        this.selectedColorName === color.name &&
        arrayIncludes(
          valuesOfObj(this.currentPalette[this.selectedColorName]),
          this.value
        )
      );
    },
    getDefaultColor(colorObj) {
      if (colorObj[this.defaultTint]) return colorObj[this.defaultTint];
      else
        return valuesOfObj(colorObj)[
          Math.round(Object.keys(colorObj).length / 2) - 1
        ]; // get the color in the middle
    }
  },
  computed: {
    colors() {
      const colors = [];
      const palette = this.subPalette
        ? this.currentPalette[this.subPalette]
        : this.currentPalette;
      const subName = this.subPalette ? this.subPalette + " - " : "";

      Object.keys(palette).forEach(key => {
        let value = palette[key];
        colors.push({
          name: subName + key,
          value: typeof value === "string" ? value : this.getDefaultColor(value)
        });
      });
      return colors;
    },
    circleStyle() {
      const padding = Math.round((this.colorSize * 30) / 100);
      const border = Math.round((this.colorSize * 8) / 100);

      return {
        padding: (padding < 1 ? 1 : padding) + "px",
        "border-width": (border < 1 ? 1 : border) + "px"
      };
    },
    currentPalette() {
      if (!this.palette) return materialPalette;
      else if (typeof this.palette === "string") {
        const availablePalettes = {
          material: materialPalette,
          "material-full": fullMaterialPalette,
          "material-accent": accentMaterialPalette
        };
        console.assert(
          arrayIncludes(Object.keys(availablePalettes), this.palette),
          "You passed in an unknown palette string. Following palettes are available:" +
            Object.keys(availablePalettes)
        );
        return availablePalettes[this.palette];
      } else return this.palette;
    },
    wrapperMinHeight() {
      const numberOfRows = Math.ceil(
        Object.keys(this.currentPalette).length / this.colorsPerRow
      );
      return (
        this.colorSize * numberOfRows +
        this.colorMargin * numberOfRows * 2 +
        "px"
      );
    },
    wrapperWidth() {
      return (
        this.colorSize * this.colorsPerRow +
        this.colorMargin * this.colorsPerRow * 2 +
        "px"
      );
    },
    colorSizePx() {
      return this.colorSize + "px";
    }
  },
  data() {
    return {
      subPalette: undefined,
      selectedColorName: undefined
    };
  },
  created() {
    if (!this.value || this.value.length !== 7 || this.selectedColorName)
      return; // value must be in hex format, eg: #ffffff
    Object.keys(this.currentPalette).forEach(key => {
      let valueOrObject = this.currentPalette[key];
      const values =
        typeof valueOrObject === "string"
          ? [valueOrObject]
          : valuesOfObj(valueOrObject);
      if (arrayIncludes(values, this.value)) this.selectedColorName = key;
    });
  }
};
</script>

<style>
.mdcp,
.mdcp * {
  box-sizing: content-box;
  text-align: left;
  line-height: 1;
  font-size: 0;
  transition: all 0.225s ease-in-out;
  -webkit-tap-highlight-color: transparent;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  -webkit-touch-callout: none;
  tap-highlight-color: transparent;
  user-select: none;
  outline-style: none;
  cursor: pointer;
}

.mdcp {
  margin: 0;
  padding: 0;
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;
  align-items: flex-start;
  align-content: flex-start;
}

.mdcp .mdcp-color {
  border-radius: 100%;
  position: relative;
  order: 1;
}

.mdcp .mdcp-back-icon {
  text-align: center;
  border-radius: 100%;
  position: relative;
}

.mdcp .mdcp-back-icon:hover {
  background: rgba(0, 0, 0, 0.19);
}

.mdcp .mdcp-color:hover {
  transform: scale(1.115);
}

.mdcp .mdcp-circle {
  position: absolute;
  border-style: solid;
  border-color: rgba(0, 0, 0, 0);
  border-radius: 100%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.mdcp .mdcp-selected .mdcp-circle {
  border-color: rgba(255, 255, 255, 1);
}

.mdcp .mdcp-selected.mdcp-is-light .mdcp-circle {
  border-color: #555555;
}

.mdcp .mdcp-circle {
  transition: all 0.4s ease-in-out;
}
</style>
