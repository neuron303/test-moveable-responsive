<template>
  <div class="main">
    <div class="container" id="container" ref="container">
      <div class="target" id="target" style="transform: translate(0px, 0px) rotate(0deg) scale(1, 1);" ref="target">
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
      <button @click="$refs.target.style.transform = ' translate(0px, 0px) rotate(0deg) scale(1, 1)'">reset</button>
      <button @click="moveLeft(50)">move 50px left from current position</button>
      <button @click="$refs.container.style.width = '25%'">set container width to 50%</button>
      <button @click="$refs.container.style.width = '50%'">set container width to 100%</button>

      {{ transform }}
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
  background-color: red;
}

.debug {
  border: 1px solid black;
  display:flex;
  flex-direction: column;
  width: 50%;
  height: 400px;
  overflow: auto;
  padding: 10px;
}
.container {
  background:url("/public/Rastergrafik.png" ) no-repeat ;
  background-size: 100% ;
  border: 1px solid black;
  width: 50%;
  max-height: 400px;
  position: relative;
  resize: both;
  overflow: auto;

}
</style>
<script setup>
import { computed, onMounted, ref } from "vue";
import Moveable from "vue3-moveable";

const target = ref(null);
const moveable = ref(null);
const onDrag = ({ transform }) => {
  target.value.style.transform = transform;
  console.log(transform);
};
const onScale = ({ transform }) => {
  target.value.style.transform = transform;

};
const onRotate = ({ transform }) => {
  target.value.style.transform = transform;

};

const targetOriginalDimension = ref({ width: 0, height: 0 });
const containerOriginalDimension = ref({ width: 0, height: 0 });
const targetOrginalTransform = ref("");
onMounted(() => {
  targetOriginalDimension.value = target.value.getBoundingClientRect();
  containerOriginalDimension.value = document.getElementById("container").getBoundingClientRect();
  //create a copy instead of reference
  targetOrginalTransform.value = {style: Object.assign({}, target.value.style)};

  let isInit = true;

  const resizeObserver = new ResizeObserver((entries) => {
    const { width, height } = entries[0].contentRect;
    if(isInit){
      isInit = false;
      return;
    }

    //get current translate values from target
    const match = target.value.style.transform.match(/(translate\(([^)]+)\))/);
    const values = match[2].split(",");
    const currentTransX = parseFloat(values[0]);
    const currentTransY = parseFloat(values[1]);
    console.log("currentTransX", currentTransX);


    //get current scale values from target
    const scale = targetOrginalTransform.value.style.transform.match(/scale\(([^)]+)\)/);
    const scaleValues = scale[1].split(",");
    //get current rotate values from target
    const rotate = targetOrginalTransform.value.style.transform.match(/rotate\(([^)]+)\)/);
    let rotateValues = rotate[1].split(",") || [0]; 

    const newScaleX =  width / containerOriginalDimension.value.width;
    const newScaleY = width /  containerOriginalDimension.value.height;

    //translate depends on scale so we need to recalculate the translate values

    const offsetX = (targetOriginalDimension.value.width * newScaleX) / 2;
    const offsetY = (targetOriginalDimension.value.height * newScaleY) / 2;
    const newTransX = currentTransX * newScaleX - offsetX;
    const newTransY = currentTransY * newScaleY - offsetY;
    
    console.log("newTransX", newTransX);
   
    

    


    document.getElementById("target").style.transform = `translate(${newTransX}px, ${newTransY}px) rotate(${rotateValues[0]}) scale(${newScaleX}, ${newScaleY})`;
    //target.value.style.transform = `translate(${newTransX}px, ${newTransY}px) rotate(${rotateValues[0]}deg) scale(${newScaleX}, ${newScaleY})`;

    //update moveable rect
    moveable.value.updateRect();
  });

  resizeObserver.observe(document.getElementById("container"))



})


computed(() => {
  transform : () => {
    return target.value.style.transform
  }
})

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
  let rotateValues = rotate[1].split(",") || [0];

  
  target.value.style.transform = `translate(${newTransX}px, ${newTransY}px) rotate(${rotateValues[0]}) scale(${scaleValues[0]}, ${scaleValues[1]})`;

  moveable.value.updateRect();

};
</script>

