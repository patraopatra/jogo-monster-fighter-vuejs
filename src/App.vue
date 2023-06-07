<script setup>
import { ref, watch } from 'vue'
import { computed } from '@vue/reactivity'

// --------------------------------------------------------- dados

const playerHealth = ref(100)
const monsterHealth = ref(100)
const roundAtual = ref(0)
const vencedor = ref(null)
const logMessages = ref([])

// --------------------------------------------------------- computed

const monsterBarStyle = computed(() => {
  if (monsterHealth.value < 0) {
    monsterHealth.value = 0
  }
  return { width: monsterHealth.value + '%' }
})

const playerBarStyle = computed(() => {
  if (playerHealth.value < 0) {
    playerHealth.value = 0
  }
  return { width: playerHealth.value + '%' }
})

const podeUsarEspecial = computed(() => {
  return roundAtual.value % 3 !== 0
})

//------------------------------------------------------------- watchers

watch(() => playerHealth.value, // qual valor vai ficar de olho
  (value) => {               // propriedade de onde vai ficar de olho
    if (value <= 0 && monsterHealth.value <= 0) {
      // Empate
      vencedor.value = 'empate'
    }
    else if (value <= 0) {
      // Player perdeu
      vencedor.value = 'monstro'
    }

  })
watch(() => monsterHealth.value, // qual valor vai ficar de olho
  (value) => {               // propriedade de onde vai ficar de olho
    if (value <= 0 && playerHealth.value <= 0) {
      // Empate
      vencedor.value = 'empate'
    }
    else if (value <= 0) {
      // Monstro perdeu
      vencedor.value = 'player'
    }

  })

//----------------------------------------------------- métodos

function random(min, max) {
  return Math.floor(Math.random() * (max - min)) + min
}

function atacarMonstro() {
  roundAtual.value++
  const attack = random(5, 12)
  monsterHealth.value -= attack // mesma coisa que monsterHealth - attack
  escreverLog('jogador', 'atacou', attack)
  atacarPlayer()
}

function atacarPlayer() {
  const attack = random(10, 15)
  playerHealth.value -= attack
  escreverLog('monstro', 'atacou', attack)
}

function atacarEspecialMonstro() {
  roundAtual.value++
  const attack = random(10, 24)
  monsterHealth.value -= attack
  escreverLog('jogador', 'especial', attack)
  atacarPlayer()
}

function curar() {
  roundAtual.value++
  const valor = random(8, 20)
  if (playerHealth.value + valor > 100) {
    playerHealth.value = 100
  }
  else {
    playerHealth.value += valor
  }
  escreverLog('jogador', 'curou', valor)
  atacarPlayer()
}

function iniciarJogo() {
  playerHealth.value = 100
  monsterHealth.value = 100
  vencedor.value = null
  roundAtual.value = 0
  logMessages.value = []
}

function entregarPartida() {
  vencedor.value = 'monstro'
}

function escreverLog(quem, oque, valor) {
  logMessages.value.unshift({
    quemFez: quem,
    oQueFez: oque,
    quanto: valor
  })
}

</script>

<template>

    <header>
      <h1>MONSTER FIGHTER</h1>
    </header>

    <div id="game">

      <section id="monster" class="container">
        <h2>Vida do Monstro</h2>
        <div class="barravida">
          <div class="valor_barravida" :style="monsterBarStyle"></div>
        </div>
      </section>

      <section id="player" class="container">
        <h2>Vida do Jogador</h2>
        <div class="barravida">
          <div class="valor_barravida" :style="playerBarStyle"></div>
        </div>
      </section>

      <section class="container" v-if="vencedor">
        <h1>Fim de jogo!</h1>
        <h2 v-if="vencedor === 'monstro'">Você perdeu!</h2>
        <h2 v-else-if="vencedor === 'player'">Você ganhou!</h2>
        <h2 v-else="vencedor === 'empate'">Empate!</h2>
        <button @click="iniciarJogo">JOGAR NOVAMENTE</button>
      </section>

      <section id="controles" v-if="!vencedor">
        <button @click="atacarMonstro">ATACAR</button>
        <button :disabled="podeUsarEspecial" @click="atacarEspecialMonstro">ATAQUE ESPECIAL</button>
        <button @click="curar">CURAR-SE</button>
        <button @click="entregarPartida">RENDER-SE</button>
      </section>

      <section id="log">
        <h2>Comandos</h2>
        <ul>
          <li v-for="logMessage in logMessages">
            <span :class="{'log--jogador': logMessage.quemFez === 'jogador', 'log--monstro': logMessage.quemFez === 'monstro'}">{{ logMessage.quemFez === 'jogador' ? 'Jogador' : 'Monstro' }}</span>
            <span v-if="logMessage.oQueFez === 'curou'"> se curou com <span class="log--curou">{{
              logMessage.quanto }}</span> </span>
            <span v-else> atacou e deu dano de <span class="log--atacou">{{ logMessage.quanto }}</span></span>
          </li>
        </ul>
      </section>

    </div>
</template>

<style scoped>
header {
  text-align: center;
  background-color: rgb(255, 94, 0);
  padding: 0.5rem;
  border-radius: 2rem;
  width: 100%;
}

h1 {
  font-size: 3rem;
  color: white;
}

h2 {
  font-size: 2rem;
  color: white;
}

section {
  width: 100%;
  margin: 1rem;
}
span{
  font-size: 1.4rem;
}

.barravida {
  width: 100%;
  height: 40px;
  border: 2px solid black;
  background-color: rgb(255, 31, 31);
}

.valor_barravida {
  background-color: rgb(16, 179, 1);
  width: 100%;
  height: 100%;
}

.container {
  text-align: center;
  padding: 0.5rem;
  border-radius: 15px;
}

#controles {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}

button {
  font: inherit;
  font-size: 2rem;
  border: 2px solid black;
  background-color: rgb(255, 0, 0);
  margin: 0.5rem;
  border-radius: 5px;
  width: 18rem;
  cursor: pointer;
  color: white;
}

button:focus {
  outline: none;
}

button:hover,
button:active {
  background-color: rgb(32, 65, 253);
  border-color: rgb(32, 65, 253);
}

button:disabled {
  background-color: #ccc;
  border-color: #ccc;
  color: #3f3f3f;
  cursor: not-allowed;
}

#game {
  display: flex;
  flex-direction: column;
  align-items: center;
}

#log ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

#log li {
  margin: 0.5rem 0;
}

.log--jogador {
  color: rgb(242, 255, 0);
}

.log--monstro {
  color: rgb(255, 0, 247);
}

.log--curou {
  color: rgb(30, 255, 0);
}

.log--atacou {
  color: red;
}
</style>
