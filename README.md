# Vue Pokédex

Small Pokédex built with **Vue 3 + TypeScript + Vite**, using the public [PokéAPI](https://pokeapi.co/).

### Features

- Search Pokémon by name or ID (e.g. `pikachu`, `1`, `25`)
- Shows official sprite, ID, and name
- Basic loading and error states

### Running locally

```bash
npm install
npm run dev

# Vue Pokédex

Small Pokédex web app built with **Vue 3 + TypeScript + Vite**, using the public [PokéAPI](https://pokeapi.co/) for data.

I built this project to get hands-on experience with Vue 3 (Composition API) for front-end roles.

---

## Features

- 🔍 **Search by name or ID**  
  Type `pikachu`, `bulbasaur`, or an ID like `1` or `25` to fetch a Pokémon.

- 🎴 **Pokémon detail card**  
  Displays the Pokémon’s name, ID, and official front sprite from PokéAPI.

- ✅ **Basic UX states**  
  - Loading indicator while data is fetched  
  - Error message when a Pokémon can’t be found

---

## Tech Stack

- [Vue 3](https://vuejs.org/) with `<script setup>` + Composition API  
- [TypeScript](https://www.typescriptlang.org/)  
- [Vite](https://vitejs.dev/) for bundling and dev server  
- Plain CSS for styling (no UI framework yet)  
- [PokéAPI](https://pokeapi.co/) for Pokémon data

---

## Running Locally

```bash
# install dependencies
npm install

# start dev server (usually http://localhost:5173)
npm run dev
```

---

## What I Practiced

- Setting up a **Vue 3 + TypeScript + Vite** project from scratch  
- Using **reactive state** with `ref()` and binding inputs with `v-model`  
- Handling **async API calls** and surfacing loading/error states in the UI  
- Keeping a small, single-component Vue app clean and readable

---

## Possible Future Enhancements

- Grid view listing multiple Pokémon with “Load more” pagination  
- Richer details (types, stats, abilities)  
- Reusable `<PokemonCard />` component  
- Filtering by type (fire, water, grass, etc.)  
- Light/dark theme toggle