<script setup lang="ts">
import { ref } from 'vue'

//search input state
const query = ref('pikachu')

//loading / error
const loading = ref(false)
const error = ref<string | null>(null)

//current Pokemon result
const pokemon = ref<null | {
  name: string
  id: number
  sprite: string
}>(null)

async function searchPokemon() {
  if (!query.value.trim()) return

  loading.value = true
  error.value = null
  pokemon.value = null

  try {
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${query.value.toLowerCase()}`)

    if (!response.ok) {
      throw new Error('Pokemon not found')
    }

    const data = await response.json()

    pokemon.value = {
      name: data.name,
      id: data.id,
      sprite: data.sprites.front_default,
    }
  } catch (err) {
    if (err instanceof Error) {
      error.value = err.message
    } else {
      error.value = 'Something went wrong'
    }
  } finally {
    loading.value = false
  }
}

// show something on first load
searchPokemon()
</script>

<template>
  <main class="app">
    <h1>Vue Pokedex</h1>
    <p>Type a Pokemon name or ID (e.d., pikachu, bulbasaur, 25) and hit search</p>

    <section class="search">
      <input
        v-model="query"
        type="text"
        placeholder="Search Pokemon"
        @keyup.enter="searchPokemon"
      />
      <button @click="searchPokemon">Search</button>
    </section>

    <section class="status">
      <p v-if="loading">Loading...</p>
      <p v-else-if="error" class="error">{{ error }}</p>
    </section>

    <section v-if="pokemon && !loading" class="card">
      <p class="id">#{{ pokemon.id }}></p>
      <img v-if="pokemon.sprite" :src="pokemon.sprite" :alt="pokemon.name" />
      <h2>{{ pokemon.name }}</h2>
    </section>
  </main>
</template>

<style scoped>
.app {
  min-height: 100vh;
  padding: 2rem;
  background: radial-gradient(circle at top, #ffebc4 0, #f0f0f0 40%);
  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    Oxygen,
    Ubuntu,
    Cantarell,
    'Open Sans',
    'Helvetica Neue',
    sans-serif;
}

h1 {
  margin-bottom: 0.25rem;
}

p {
  margin: 0.25rem 0;
}

.search {
  margin-top: 1.5rem;
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

input {
  padding: 0.5rem 0.75rem;
  font-size: 1rem;
  border-radius: 0.5rem;
  border: 1px solid #ccc;
  min-width: 220px;
}

button {
  padding: 0.5rem 1rem;
  font-size: 1rem;
  border-radius: 0.5rem;
  border: none;
  cursor: pointer;
  background-color: #ff5350;
  color: white;
  font-weight: 600;
}

button.hover {
  opacity: 0.9;
}

.status {
  height: 1.5rem;
  margin-top: 0.5rem;
}

.error {
  color: #b00020;
}

.card {
  margin-top: 2rem;
  padding: 1.5rem;
  max-width: 260px;
  border-radius: 1rem;
  background: white;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  text-align: center;
}

.card .id {
  font-size: 0.85rem;
  color: #888;
}

.card img {
  width: 120px;
  height: 120px;
  image-rendering: pixelated;
}
</style>
