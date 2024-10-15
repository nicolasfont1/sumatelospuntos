<script setup>
import { ref } from 'vue';

const pointBox = {
  left: true,
  top: false,
  right: false,
  bottom: false,
  center: false
};

const nosPoints = ref([]);
const ellosPoints = ref([]);

const addPoint = (teamPointsObject) => {
  if (!teamPointsObject.length) {
    addNewBox(teamPointsObject)
  } else {
    const lastBox = teamPointsObject[teamPointsObject.length - 1];

    if (!lastBox.top) {
      lastBox.top = true;
    } else if (!lastBox.right) {
      lastBox.right = true;
    } else if (!lastBox.bottom) {
      lastBox.bottom = true;
    } else if (!lastBox.center) {
      lastBox.center = true;
    } else {
      addNewBox(teamPointsObject);
    }
  }
};

const addNewBox = (teamPointsObject) => {
  teamPointsObject.push({ ...pointBox })
};

const removePoint = (teamPointsObject) => {
  if (!teamPointsObject.length) {
    return
  } else {
    const lastBox = teamPointsObject[teamPointsObject.length - 1];

    if (lastBox.center) {
      lastBox.center = false;
    } else if (lastBox.bottom) {
      lastBox.bottom = false;
    } else if (lastBox.right) {
      lastBox.right = false;
    } else if (lastBox.top) {
      lastBox.top = false;
    } else {
      removeLastBox(teamPointsObject)
    }
  }
};

const removeLastBox = (teamPointsObject) => {
  teamPointsObject.pop()
};

</script>

<template>
  <main class="main-container">
    <div class="team-names">
      <div class="team-nos">
        <span class="team-name">NOS</span>
      </div>
      <div class="team-ellos">
        <span class="team-name">ELLOS</span>
      </div>
    </div>
    <div class="team-points">
      <div class="buttons-container">
        <button class="point-button" @click="addPoint(nosPoints)">+</button>
        <button class="point-button" @click="removePoint(nosPoints)">-</button>
      </div>
      <div class="points-container nos">
        <div v-for="(fivePoints, index) in nosPoints" :key="index" class="five-point-box"
          :class="{ 'left-stick': fivePoints.left, 'top-stick': fivePoints.top, 'right-stick': fivePoints.right, 'bottom-stick': fivePoints.bottom, 'center-stick': fivePoints.center }">
        </div>
      </div>
      <!-- HALF TABLE -->
      <div class="points-container ellos">
        <div v-for="(fivePoints, index) in ellosPoints" :key="index" class="five-point-box"
          :class="{ 'left-stick': fivePoints.left, 'top-stick': fivePoints.top, 'right-stick': fivePoints.right, 'bottom-stick': fivePoints.bottom, 'center-stick': fivePoints.center }">
        </div>
      </div>
      <div class="buttons-container">
        <button class="point-button" @click="addPoint(ellosPoints)">+</button>
        <button class="point-button" @click="removePoint(ellosPoints)">-</button>
      </div>
    </div>
  </main>
</template>
