<script setup lang="ts">
import { ref, onMounted } from 'vue'

//search input state
const query = ref('pikachu')

//loading / error
const loading = ref(false)
const error = ref<string | null>(null)

type PokemonSummary = {
  name: string
  id: number
  sprite: string
}

type PokemonDetail = {
  name: string
  id: number
  sprite: string
  stats: { name: string; baseStat: number }[]
  abilities: string[]
}

//current Pokemon result (detail card)
const pokemon = ref<PokemonDetail | null>(null)

//grid state (summaries only)
const pokemonList = ref<PokemonSummary[]>([])
const listLoading = ref(false)
const listError = ref<string | null>(null)
const pageSize = 24
const nextOffset = ref(0)

function normalizeQuery(value: string): string {
  return value.trim().toLowerCase().replace(/^#/, '')
}

async function searchPokemon() {
  const normalizedQuery = normalizeQuery(query.value)
  if (!normalizedQuery) return

  loading.value = true
  error.value = null
  pokemon.value = null

  try {
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${normalizedQuery}`)

    if (!response.ok) {
      throw new Error('Pokemon not found')
    }

    type ApiStat = { base_stat: number; stat: { name: string } }
    type ApiAbility = { ability: { name: string } }

    const data = (await response.json()) as {
      name: string
      id: number
      sprites: { front_default: string }
      stats: ApiStat[]
      abilities: ApiAbility[]
    }

    pokemon.value = {
      name: data.name,
      id: data.id,
      sprite: data.sprites.front_default,
      stats: data.stats.map((s) => ({
        name: s.stat.name,
        baseStat: s.base_stat,
      })),
      abilities: data.abilities.map((a) => a.ability.name),
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

async function loadPokemonPage() {
  listLoading.value = true
  listError.value = null

  try {
    const response = await fetch(
      `https://pokeapi.co/api/v2/pokemon?limit=${pageSize}&offset=${nextOffset.value}`,
    )

    if (!response.ok) {
      throw new Error('Failed to load Pokemon list')
    }

    const data = (await response.json()) as { results: { name: string; url: string }[] }

    const newPokemons: PokemonSummary[] = data.results.map((item) => {
      const match = item.url.match(/\/pokemon\/(\d+)\//)
      const id = match ? Number(match[1]) : NaN

      const sprite = Number.isFinite(id)
        ? `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${id}.png`
        : ''

      return {
        name: item.name,
        id,
        sprite,
      }
    })

    pokemonList.value = [...pokemonList.value, ...newPokemons]
    nextOffset.value += pageSize
  } catch (err) {
    if (err instanceof Error) {
      listError.value = err.message
    } else {
      listError.value = 'Something went wrong loading the list'
    }
  } finally {
    listLoading.value = false
  }
}

function selectFromGrid(p: PokemonSummary) {
  query.value = p.name
  void searchPokemon()
}

onMounted(() => {
  // show something on first load
  searchPokemon()
  loadPokemonPage()
})
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
      <p class="id">#{{ pokemon.id }}</p>
      <img v-if="pokemon.sprite" :src="pokemon.sprite" :alt="pokemon.name" />
      <h2 class="name">{{ pokemon.name }}</h2>

      <div v-if="pokemon.stats.length" class="stats">
        <h3>Stats</h3>
        <ul>
          <li v-for="stat in pokemon.stats" :key="stat.name">
            <span class="stat-name">{{ stat.name }}</span>
            <span class="stat-value">{{ stat.baseStat }}</span>
          </li>
        </ul>
      </div>

      <div v-if="pokemon.abilities.length" class="abilities">
        <h3>Abilities</h3>
        <ul>
          <li v-for="ability in pokemon.abilities" :key="ability">
            {{ ability }}
          </li>
        </ul>
      </div>
    </section>

    <section class="grid-section">
      <div class="grid-header">
        <h2>Pokemon Grid</h2>
        <p class="grid-meta">Showing {{ pokemonList.length }} Pokemon</p>
      </div>

      <p v-if="listError" class="error">
        {{ listError }}
      </p>

      <div v-if="listLoading && pokemonList.length === 0" class="grid-loading">
        Loading Pokemon...
      </div>

      <div v-if="pokemonList.length" class="grid">
        <article
          v-for="p in pokemonList"
          :key="p.id"
          class="card card--grid"
          @click="selectFromGrid(p)"
        >
          <p class="id">#{{ p.id }}</p>
          <img v-if="p.sprite" :src="p.sprite" :alt="p.name" />
          <h3 class="name">{{ p.name }}</h3>
        </article>
      </div>

      <div class="grid-actions">
        <button type="button" class="load-more" :disabled="listLoading" @click="loadPokemonPage">
          {{ listLoading ? 'Loading…' : 'Load more' }}
        </button>
      </div>
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
  color: #222;
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

button:hover {
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
  color: #555;
}

.card img {
  width: 120px;
  height: 120px;
  image-rendering: pixelated;
}

.stats {
  margin-top: 1rem;
  text-align: left;
}

.stats h3,
.abilities h3 {
  font-size: 0.95rem;
  margin-bottom: 0.25rem;
}

.stats ul,
.abilities ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.stats li {
  display: flex;
  justify-content: space-between;
  font-size: 0.9rem;
}

.stat-name {
  text-transform: capitalize;
}

.abilities {
  margin-top: 0.75rem;
  text-align: left;
}

.abilities li {
  font-size: 0.9rem;
  text-transform: capitalize;
}

.grid-section {
  margin-top: 3rem;
}

.grid-header {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 1rem;
}

.grid-meta {
  font-size: 0.9rem;
  color: #444;
}

.grid-loading {
  margin-top: 1rem;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
  gap: 1rem;
  margin-top: 1rem;
}

.card--grid {
  cursor: pointer;
  max-width: none;
}

.card--grid .name {
  text-transform: capitalize;
}

.grid-actions {
  margin-top: 1.5rem;
  display: flex;
  justify-content: center;
}

.load-more {
  padding: 0.5rem 1.25rem;
  border-radius: 999px;
  border: none;
  cursor: pointer;
  background-color: #2b7cff;
  color: #fff;
  font-weight: 600;
}

.load-more:disabled {
  opacity: 0.6;
  cursor: default;
}
</style>
