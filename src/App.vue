<script setup>
import { ref, nextTick, onMounted } from 'vue';
import confetti from 'canvas-confetti';

onMounted(() => {
  const savedPoints = localStorage.getItem('savedPoints');
  const savedTeamAliases = localStorage.getItem('savedAliases');

  if (savedPoints) { loadSavedPoints(savedPoints); }
  if (savedTeamAliases) { loadSavedAliases(savedTeamAliases); }
})

const loadSavedPoints = (savedPoints) => {
  const pointsToLoad = JSON.parse(savedPoints);

  for (let i = 0; i < pointsToLoad.nos; i++) {
    addPoint(nosPoints.value, 'NOS');
  };

  for (let i = 0; i < pointsToLoad.ellos; i++) {
    addPoint(ellosPoints.value, 'ELLOS');
  };
};

const loadSavedAliases = (savedTeamAliases) => {
  const aliasesToLoad = JSON.parse(savedTeamAliases);

  nosAlias.value = aliasesToLoad.nos;
  ellosAlias.value = aliasesToLoad.ellos;
};

const savePoints = (nosPoints, ellosPoints) => {
  const matchPoints = {
    nos: nosPoints,
    ellos: ellosPoints
  };

  localStorage.setItem('savedPoints', JSON.stringify(matchPoints));
};

const pointBox = {
  left: true,
  top: false,
  right: false,
  bottom: false,
  center: false
};

const nosPoints = ref([]);
const nosCounter = ref(0)

const ellosPoints = ref([]);
const ellosCounter = ref(0)

const winnerTeam = ref('')

const addPoint = (teamPointsObject, teamName) => {
  const lastBox = teamPointsObject[teamPointsObject.length - 1];
  if (teamPointsObject.length === 6 && lastBox.center) {
    return
  }

  if (teamName === 'NOS') {
    nosCounter.value ++
  } else {
    ellosCounter.value ++
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
        winnerTeam.value = teamName === 'NOS' ? (nosAlias.value || teamName) : (ellosAlias.value || teamName);
        throwConfetti();
        return
      };
      lastBox.center = true;
    } else {
      addNewBox(teamPointsObject);
    }
  }

  savePoints(nosCounter.value, ellosCounter.value);
};

const addNewBox = (teamPointsObject) => {
  teamPointsObject.push({ ...pointBox })
};

const removePoint = (teamPointsObject, teamName) => {
  if (!teamPointsObject.length) {
    return
  } else {
    const lastBox = teamPointsObject[teamPointsObject.length - 1];

    if (teamName === 'NOS') {
      nosCounter.value --
    } else {
      ellosCounter.value --
    }

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

  savePoints(nosCounter.value, ellosCounter.value);
};

const removeLastBox = (teamPointsObject) => {
  teamPointsObject.pop()
};

const reset = () => {
  nosPoints.value = [];
  nosCounter.value = 0;
  ellosPoints.value = [];
  ellosCounter.value = 0;
  winnerTeam.value = '';

  savePoints(nosCounter.value, ellosCounter.value);
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

const editNos = ref(false);
const inputNos = ref(null);
const nosAlias = ref('');

const editEllos = ref(false);
const inputEllos = ref(null);
const ellosAlias = ref('');

const onInputFocus = (team) => {
  if (team === 'nos') {
    editNos.value = true;
    nextTick(() => {
      inputNos.value?.focus();
    });
  } else if (team === 'ellos') {
    editEllos.value = true;
    nextTick(() => {
      inputEllos.value?.focus();
    });
  }
};

const saveAliases = (team) => {
  // I need to make an object to access dinamically editNos or editEllos states
  const editStates = {
    nos: editNos,
    ellos: editEllos
  }

  editStates[team].value = false;

  const teamAliases = {
    nos: nosAlias.value,
    ellos: ellosAlias.value
  }

  localStorage.setItem('savedAliases', JSON.stringify(teamAliases));
};

const throwConfetti = () => {
  confetti({
    particleCount: 400,
    spread: 50,
    decay: 0.9,
    gravity: .8,
    origin: {
      y: 0.7
    },
    ticks: 200
  })
}
</script>

<template>
  <main class="main-container">
    <section class="game-container">
      <div class="team-names">
        <span class="team-point-counter">{{nosCounter}}</span>
        <div class="team-nos">
          <span v-if="!editNos" class="team-name" @click="onInputFocus('nos')">{{ nosAlias || 'NOS' }}</span>
          <textarea v-else name="nosAliasTextArea" type="text" v-model="nosAlias" @blur="saveAliases('nos')" @keydown.enter="saveAliases('nos')" class="textarea-name" ref="inputNos"></textarea>
        </div>
        <div class="team-ellos">
          <span v-if="!editEllos" class="team-name"  @click="onInputFocus('ellos')">{{ ellosAlias || 'ELLOS' }}</span>
          <textarea v-else name="ellosAliasTextArea" type="text" v-model="ellosAlias" @blur="saveAliases('ellos')" @keydown.enter="saveAliases('ellos')" class="textarea-name" ref="inputEllos"></textarea>
        </div>
        <span class="team-point-counter">{{ellosCounter}}</span>
      </div>
      <div class="divider"></div>
      <div class="team-points">
        <div class="buttons-container">
          <button class="point-button" @click="addPoint(nosPoints, 'NOS')">+</button>
          <button class="point-button" @click="removePoint(nosPoints, 'NOS')">-</button>
          <button class="restart-button" @click="reset">
            <svg xmlns="http://www.w3.org/2000/svg" fill="#f5f5f5" viewBox="-7.5 0 32 32" version="1.1">
              <path d="M15.88 13.84c-1.68-3.48-5.44-5.24-9.040-4.6l0.96-1.8c0.24-0.4 0.080-0.92-0.32-1.12-0.4-0.24-0.92-0.080-1.12 0.32l-1.96 3.64c0 0-0.44 0.72 0.24 1.040l3.64 1.96c0.12 0.080 0.28 0.12 0.4 0.12 0.28 0 0.6-0.16 0.72-0.44 0.24-0.4 0.080-0.92-0.32-1.12l-1.88-1.040c2.84-0.48 5.8 0.96 7.12 3.68 1.6 3.32 0.2 7.32-3.12 8.88-1.6 0.76-3.4 0.88-5.080 0.28s-3.040-1.8-3.8-3.4c-0.76-1.6-0.88-3.4-0.28-5.080 0.16-0.44-0.080-0.92-0.52-1.080-0.4-0.080-0.88 0.16-1.040 0.6-0.72 2.12-0.6 4.36 0.36 6.36s2.64 3.52 4.76 4.28c0.92 0.32 1.84 0.48 2.76 0.48 1.24 0 2.48-0.28 3.6-0.84 4.16-2 5.92-7 3.92-11.12z"/>
            </svg>
          </button>
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
          <button class="point-button" @click="removePoint(ellosPoints, 'ELLOS')">-</button>
        </div>
      </div>
    </section>
    <button class="share-button" @click="shareApp">Compartir app</button>
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
      <button class="reset-button" @click="reset">Reiniciar</button>
      <button class="see-points-button" @click="winnerTeam = ''">Ver puntos</button>
    </div>
  </div>
</template>
