<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import KeyPad from '@/components/KeyPad.vue'
import ResultsTable from '@/components/ResultsTable.vue'
import ScoreBoard from '@/components/ScoreBoard.vue'
import { useGameStore } from '@/stores/game'
import { socket } from '@/services/socket'

const store = useGameStore()

const nom = ref('')

const tecles = ['A', 'S', 'D', 'F']

const tempsObjectiu = {
  A: 2,
  S: 4,
  D: 5,
  F: 7
}

const jugant = ref(false)
const resultats = ref([])

let tempsInici = 0

function entrar() {
  if (nom.value.trim() === '') return

  store.guardarNom(nom.value.trim())
}

function iniciarPartida() {
  resultats.value = []
  tempsInici = performance.now()
  jugant.value = true
}

function rebreTecla(tecla) {
  if (!jugant.value) return

  if (resultats.value.some(r => r.tecla === tecla)) return

  const tempsReal = (performance.now() - tempsInici) / 1000
  const objectiu = tempsObjectiu[tecla]
  const error = Math.abs(tempsReal - objectiu)

  resultats.value.push({
    tecla: tecla,
    objectiu: objectiu,
    tempsReal: Number(tempsReal.toFixed(2)),
    error: Number(error.toFixed(2))
  })

  if (resultats.value.length === tecles.length) {
    jugant.value = false

    // Calcular l'error total amb reduce()
    const errorTotal = resultats.value.reduce((total, r) => {
      return total + Math.abs(r.error)
    }, 0)

    const errorFinal = Number(errorTotal.toFixed(2))

    // Guardar partida en nuestro historial local
    store.guardarPartida(errorFinal)

    // Enviar resultado al servidor
    socket.emit('resultat', {
      nom: store.nomJugador,
      errorTotal: errorFinal
    })
  }
}

// CONEXIÓN CON SOCKET.IO

function connectat() {
  console.log('Connectat al servidor!', socket.id)
}

// RECIBIR CLASIFICACIÓN

function rebreClassificacio(dades) {
  console.log('Classificació rebuda:', dades)
  store.actualitzarClassificacio(dades)
}

onMounted(() => {
  socket.on('connect', connectat)
  socket.on('classificacio', rebreClassificacio)
  socket.connect()
})

onUnmounted(() => {
  socket.off('connect', connectat)
  socket.off('classificacio', rebreClassificacio)
  socket.disconnect()
})
</script>

<template>
  <main>
    <h1>TIME ATTACK</h1>

    <div v-if="!store.nomJugador">
      <h2>Benvingut a TIME ATTACK!</h2>

      <input
        v-model="nom"
        placeholder="Introdueix el teu nom"
      />

      <button @click="entrar">
        Entrar al joc
      </button>
    </div>

    <div v-else>
      <h2>Hola, {{ store.nomJugador }}!</h2>

      <ScoreBoard />

      <button v-if="!jugant && resultats.length === 0" @click="iniciarPartida">
        Començar partida
      </button>

      <KeyPad
        :tecles="tecles"
        :bloquejat="!jugant"
        :premudes="resultats.map(r => r.tecla)"
        @premuda="rebreTecla"
      />

      <p v-if="jugant">
        Prem les tecles quan creguis que ha arribat el moment!
      </p>

      <button v-if="!jugant && resultats.length === tecles.length" @click="iniciarPartida">
        Tornar a jugar
      </button>

      <ResultsTable
        v-if="!jugant && resultats.length === tecles.length"
        :resultats="resultats"
      />
    </div>
  </main>
</template>