<script setup>
import { ref, nextTick, onMounted } from 'vue';

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
  window.location.reload();
  // Useless
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
</script>

<template>
  <main class="main-container">
    <section class="game-container">
      <div class="team-names">
        <span class="team-point-counter">{{nosCounter}}</span>
        <div class="team-nos">
          <span v-if="!editNos" class="team-name" @click="onInputFocus('nos')">{{ nosAlias || 'NOS' }}</span>
          <input v-else type="text" v-model="nosAlias" @blur="saveAliases('nos')" @keydown.enter="saveAliases('nos')" class="input-name" ref="inputNos">
        </div>
        <div class="team-ellos">
          <span v-if="!editEllos" class="team-name"  @click="onInputFocus('ellos')">{{ ellosAlias || 'ELLOS' }}</span>
          <input v-else type="text" v-model="ellosAlias" @blur="saveAliases('ellos')" @keydown.enter="saveAliases('ellos')" class="input-name" ref="inputEllos">
        </div>
        <span class="team-point-counter">{{ellosCounter}}</span>
      </div>
      <div class="divider"></div>
      <div class="team-points">
        <div class="buttons-container">
          <button class="point-button" @click="addPoint(nosPoints, 'NOS')">+</button>
          <button class="point-button" @click="removePoint(nosPoints, 'NOS')">-</button>
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
