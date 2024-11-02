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

const winnerTeam = ref('')

const addPoint = (teamPointsObject, teamName) => {
  const lastBox = teamPointsObject[teamPointsObject.length - 1];
  if (teamPointsObject.length === 6 && lastBox.center) {
    return
  }

  if (!teamPointsObject.length) {
    addNewBox(teamPointsObject)
  } else {
    if (!lastBox.top) {
      lastBox.top = true;
    } else if (!lastBox.right) {
      lastBox.right = true;
    } else if (!lastBox.bottom) {
      lastBox.bottom = true;
    } else if (!lastBox.center) {
      if (teamPointsObject.length === 6) {
        lastBox.center = true;
        winnerTeam.value = teamName;
        return
      };
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

const reset = () => {
  window.location.reload();
};

const shareApp = () => {
  const URL = 'https://sumatelospuntos.vercel.app/'

  navigator.clipboard.writeText(URL)
    .then(() => {
      if (copiedURL.value) return
      copiedURL.value = true;
      setTimeout(() => {
        copiedURL.value = false;
      }, 3000);
    })
    .catch(err => {
      if (nonCopiedURL.value) return
      nonCopiedURL.value = true;
      setTimeout(() => {
        nonCopiedURL.value = false;
      }, 3000);
    });
};

const copiedURL = ref(false)
const nonCopiedURL = ref(false)
</script>

<template>
  <main class="main-container">
    <section class="game-container">
      <div class="team-names">
        <div class="team-nos">
          <span class="team-name">NOS</span>
        </div>
        <div class="team-ellos">
          <span class="team-name">ELLOS</span>
        </div>
      </div>
      <div class="divider"></div>
      <div class="team-points">
        <div class="buttons-container">
          <button class="point-button" @click="addPoint(nosPoints, 'NOS')">+</button>
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
          <button class="point-button" @click="addPoint(ellosPoints, 'ELLOS')">+</button>
          <button class="point-button" @click="removePoint(ellosPoints)">-</button>
        </div>
      </div>
    </section>
    <button class="share-button" @click="shareApp()">Compartir app</button>
  </main>
    <!-- Modal -->
  <div v-if="winnerTeam.length || copiedURL || nonCopiedURL" class="modal-outside">
    <div v-if="copiedURL" class="modal-inside">
      <span style="margin-bottom: 15px; font-size: 25px;">¡URL copiada!</span>
      <span>Pegala donde quieras para</span>
      <span>compartir la app</span>
    </div>
    <div v-if="nonCopiedURL" class="modal-inside">
      <span style="margin-bottom: 15px; font-size: 25px;">Algo salió mal</span>
      <span>No se pudo copiar la URL</span>
    </div>
    <div v-if="winnerTeam.length" class="modal-inside">
      <span style="font-size: 70px;">🏆</span>
      <span style="font-size: 30px;">Ganador</span>
      <span style="font-size: 50px; font-weight: 700;">{{ winnerTeam  }}</span>
      <button class="reset-button" @click="reset()">Reiniciar</button>
      <button class="see-points-button" @click="winnerTeam = ''">Ver puntos</button>
    </div>
  </div>
</template>
