<template>
  <div class="main">
    <div class="container" id="container" ref="container">
      <div class="target" style="transform: translate(0px, 0px) rotate(0deg) scale(1, 1);" ref="target">
        TEST
      </div>
      <Moveable
        ref="moveable"
        className="moveable"
        v-bind:target="['.target']"
        v-bind:draggable="true"
        v-bind:scalable="true"
        v-bind:rotatable="true"
        @drag="onDrag"
        @scale="onScale"
        @rotate="onRotate"
      />
    </div>

    <div class="debug">
      <button @click="$refs.target.style.transform = ' translate(0px, 10px) rotate(0deg) scale(1, 1)'">reset</button>
      <button @click="moveLeft(50)">move 50px left from current position</button>
      <button @click="$refs.container.style.width = '25%'">set container width to 50%</button>
      <button @click="$refs.container.style.width = '50%'">set container width to 100%</button>
    </div>
  </div>
</template>
<style>
.main {
  display: flex;
  justify-content: center;
}

.target {
  width: 50px;
  height: 50px;
}

.debug {
  border: 1px solid black;
  width: 50%;
  height: 400px;
  overflow: auto;
  padding: 10px;
}
.container {
  border: 1px solid black;
  width: 50%;
  height: 400px;
  position: relative;
  resize: both;
  overflow: auto;

}
</style>
<script setup>
import { onMounted, ref } from "vue";
import Moveable from "vue3-moveable";

const target = ref(null);
const moveable = ref(null);
let currentTargetTransform = "";
const onDrag = ({ transform }) => {
  target.value.style.transform = transform;
  currentTargetTransform = transform;

};
const onScale = ({ transform }) => {
  target.value.style.transform = transform;
  currentTargetTransform = transform;

};
const onRotate = ({ transform }) => {
  target.value.style.transform = transform;
  currentTargetTransform = transform;

};

const targetOriginalDimension = ref({ width: 0, height: 0 });
const containerOriginalDimension = ref({ width: 0, height: 0 });

onMounted(() => {
  targetOriginalDimension.value = target.value.getBoundingClientRect();
  containerOriginalDimension.value = document.getElementById("container").getBoundingClientRect();
})

window.addEventListener("resize", () => {


  const { width, height, left, top } = document
    .getElementById("container")
    .getBoundingClientRect();

  const zoomFactor = width / containerOriginalDimension.value.width;

  
  resize(zoomFactor, zoomFactor);
});

const resize = (elScaleFactor, cntScaleFactor) => {
  console.log("elScaleFactor", elScaleFactor, "cntScaleFactor", cntScaleFactor);
  // get scale from target
  const scaleMatch = currentTargetTransform.match(
    /(translate\(([^)]+)\))/
  );
  const scaleValues = scaleMatch[2].split(",");
  //const scale = target.value.style.transform.match(/scale\(([^)]+)\)/);
  let scaleX = scaleValues === null ? 1 : parseFloat(scaleValues[0]);
  let scaleY = scaleValues === null ? 1 : parseFloat(scaleValues[1]);

  const newScaleX = elScaleFactor;
  const newScaleY = elScaleFactor;
  console.log(scaleMatch, scaleValues, scaleX, scaleY);

  // get current translation
  const translate = currentTargetTransform.match(
    /(translate\(([^)]+)\))/
  )

  const values = translate[2].split(",");
  const currentTransX = parseFloat(values[0]);
  const currentTransY = parseFloat(values[1]);
  console.log("currentTransX", currentTransX, "currentTransY", currentTransY);

  //const { width: elWidth, height: elHeight } = orginalDimension.value;
  const { width: elWidth, height: elHeight } = target.value.getBoundingClientRect();

  const originalOffsetX = elWidth / elScaleFactor;
  const originalOffsetY = elHeight / elScaleFactor;
  console.log("originalOffsetX", originalOffsetX, "originalOffsetY", originalOffsetY);
  const currentOffsetX = currentTransX + originalOffsetX;
  const currentOffsetY = currentTransY + originalOffsetY;
  target.value.style.transform = `translate(${currentOffsetX}px, ${currentOffsetY}px) scale(${newScaleX}, ${newScaleY})`;

  moveable.value.updateRect();

  //console.log(newTransX, newTransY, currentWidth, currentHeight);
};

const moveLeft = (px) => {
  const transform = target.value.style.transform;

  
  const match = transform.match(/(translate\(([^)]+)\))/);
  const values = match[2].split(",");
  const currentTransX = parseFloat(values[0]);
  const currentTransY = parseFloat(values[1]);
  const newTransX = currentTransX - px;
  const newTransY = currentTransY;

  const scale = target.value.style.transform.match(/scale\(([^)]+)\)/);
  const scaleValues = scale[1].split(",");

  const rotate = target.value.style.transform.match(/rotate\(([^)]+)\)/);
  const rotateValues = rotate[1].split(",");
  
  target.value.style.transform = `translate(${newTransX}px, ${newTransY}px) rotate(${rotateValues[0]}) scale(${scaleValues[0]}, ${scaleValues[1]})`;

  moveable.value.updateRect();

};
</script>
