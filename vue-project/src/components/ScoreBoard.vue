<script setup>
import { computed } from 'vue'
import { useGameStore } from '@/stores/game'

const store = useGameStore()

const millorMarca = computed(() => {
  if (store.partides.length === 0) {
    return null
  }

  return Math.min(...store.partides.map(partida => partida.errorTotal))
})

// Ordenar la clasificación de menor a mayor error
const classificacioOrdenada = computed(() => {
  return [...store.classificacio].sort((a, b) => {
    return a.errorTotal - b.errorTotal
  })
})
</script>

<template>
  <div>
    <h2>HISTORIAL DE PARTIDES</h2>

    <p v-if="store.partides.length === 0">
      Encara no hi ha cap partida.
    </p>

    <div v-else>
      <h3>Millor marca: {{ millorMarca }} s</h3>

      <table>
        <thead>
          <tr>
            <th>Partida</th>
            <th>Jugador</th>
            <th>Error total</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="partida in store.partides"
            :key="partida.id"
          >
            <td>{{ partida.id }}</td>
            <td>{{ partida.nom }}</td>
            <td>{{ partida.errorTotal }} s</td>
          </tr>
        </tbody>
      </table>

      <button @click="store.esborrarPartides()">
        Esborrar historial
      </button>
    </div>

    <hr>

    <h2>CLASSIFICACIÓ GLOBAL</h2>

    <p v-if="store.classificacio.length === 0">
      Encara no hi ha resultats. Sigues el primer a jugar!
    </p>

    <table v-else>
      <thead>
        <tr>
          <th>Posició</th>
          <th>Jugador</th>
          <th>Error total</th>
        </tr>
      </thead>

      <tbody>
        <tr
          v-for="(resultat, index) in classificacioOrdenada"
          :key="index"
          :style="index === 0 ? 'background-color: lightgreen; font-weight: bold;' : ''"
        >
          <td>{{ index + 1 }}</td>
          <td>
            {{ index === 0 ? '🏆 ' : '' }}{{ resultat.nom }}
          </td>
          <td>{{ resultat.errorTotal }} s</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>