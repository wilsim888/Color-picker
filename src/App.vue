<script lang="ts">

import { defineComponent, ref, reactive, toRefs, computed } from "vue";

interface Color {
  r: number;
  g: number;
  b: number;
}

export default defineComponent({
  name: "ColorPicker",
  setup() {
    //tableau react pour les colors
    const savedColors = ref<Color[]>([]);

    //en mode modifier ou pas
    const isEditing = ref<boolean>(false);

    // index des couleur selectionnées
    const selectedColorIndex = ref<number | null>(null);

    // objet couleur
    const colors = reactive<Color>({ r: 0, g: 0, b: 0 });

    // propriétés couleur --> ref
    const colorRefs = toRefs(colors);
    const currentSelectedIndex = ref<number | null>(null);

    //calcul la couleur
    const colorStyle = computed(() => {
      return `rgb(${colors.r}, ${colors.g}, ${colors.b})`;
    });

    // triage des couleurs
    const sortedColors = computed(() => {
      return [...savedColors.value].sort((a, b) => colorIntensity(a) - colorIntensity(b));
    });
//calcul nb couleur
    const colorCount = computed(() => savedColors.value.length);

    const isColorSaved = computed(() =>
      savedColors.value.some(
        (savedColor) => savedColor.r === colors.r && savedColor.g === colors.g && savedColor.b === colors.b
      )
    );
//fct pour sauver
    function saveColor() {
      if (isEditing.value && selectedColorIndex.value !== null) {
        savedColors.value[selectedColorIndex.value] = { ...colors };
        isEditing.value = false;
        selectedColorIndex.value = null;
      } else {
        savedColors.value.push({ ...colors });
      }

      colors.r = 0;
      colors.g = 0;
      colors.b = 0;
      currentSelectedIndex.value = null;
      isEditing.value = false;
      selectedColorIndex.value = null;
    }
// fct pour modifier
    function editColor(index: number) {
      if (currentSelectedIndex.value === index) {
        currentSelectedIndex.value = null;
        isEditing.value = false;
        selectedColorIndex.value = null;
      } else {
        const colorToEdit = savedColors.value[index];
        colors.r = colorToEdit.r;
        colors.g = colorToEdit.g;
        colors.b = colorToEdit.b;
        isEditing.value = true;
        selectedColorIndex.value = index;
        currentSelectedIndex.value = index;
      }
    }
// fct pour reset
    function resetColor() {
      colors.r = 0;
      colors.g = 0;
      colors.b = 0;
      savedColors.value = [];
      isEditing.value = false;
      selectedColorIndex.value = null;
      currentSelectedIndex.value = null;
    }
// calcule l'intensité chromatique
    function colorIntensity(color: Color): number {
      return color.r + color.g + color.b;
    }

    return {
      colorCount,
      colorStyle,
      savedColors,
      currentSelectedIndex,
      sortedColors,
      saveColor,
      editColor,
      resetColor,
      isColorSaved,
      isEditing,
      ...colorRefs,
    };
  },
});
</script>

<template>
  <div id="App">
    <header>
      <p>Fabulous Color Picker</p>
    </header>
    <div>
      <label for="colorCount">Nombre de couleurs : {{ colorCount }}</label>
    </div>
    <hr />
    <div>
      <div :style="{ backgroundColor: colorStyle }" class="color-display"></div>
    </div>
    <hr />
    <div class="sliders">
      <div>
        <label for="r">R</label>
        <input type="range" id="r" v-model.number="r" min="0" max="255" />
      </div>
      <div>
        <label for="g">G</label>
        <input type="range" id="g" v-model.number="g" min="0" max="255" />
      </div>
      <div>
        <label for="b">B</label>
        <input type="range" id="b" v-model.number="b" min="0" max="255" />
      </div>
    </div>
    <hr />
    <button @click="saveColor" :disabled="isColorSaved">{{ isEditing ? "Modifier" : "Sauvegarder" }}</button>
    <button @click="resetColor">Reset</button>
    <div class="saved-colors">
      <div
        v-for="(color, index) in sortedColors"
        :key="index"
        class="saved-color"
        :class="{ 'selected-color': currentSelectedIndex === index }"
        :style="{ backgroundColor: `rgb(${color.r}, ${color.g}, ${color.b})` }"
        @click="editColor(index)"
      >
        <span v-if="currentSelectedIndex === index"> R: {{ r }}, G: {{ g }}, B: {{ b }} </span>
      </div>
    </div>
    <hr />
    <footer>
      <div v-if="savedColors.length > 0">
        <div v-for="(savedColor, index) in savedColors" :key="`footer-${index}`">
          <div
            class="footer-color"
            :style="{ backgroundColor: `rgb(${savedColor.r}, ${savedColor.g}, ${savedColor.b})` }"
          ></div>
        </div>
      </div>
      <div v-else>
        <p>Bienvenue au Fabulous Color Picker</p>
      </div>
    </footer>
  </div>
</template>






<style scoped>
.color-display {
  width: 140px;
  height: 140px;
  border: 2px solid #000;
  margin: auto;
  display: block;
  border-radius: 15px;
  box-shadow: 0 0 10px #020202;
}

.saved-color {
  width: 70px;
  height: 70px;
  margin: 2px;
  display: inline-block;
  cursor: pointer;
  border: 4px solid #000;
  border-radius: 15px;
  position: relative;
  transition: transform 0.3s, box-shadow 0.3s;
}
.saved-color.selected-color {
  transform: scale(1.3);
  box-shadow: 0 0 10px #ffff00;
}

.saved-color span {
  position: absolute;
  font-size: 0.4rem;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: rgb(7, 8, 1);
}
#app button:disabled {
  background-color: #f497b3;
  border-color: #f497b3;
  cursor: not-allowed;
  opacity: 0.5;
}
</style>
