<template>
  <div class="card p-4 shadow border-0 animate__animated animate__fadeIn">
      <router-link to="/" class="btn btn-sm btn-outline-secondary mb-3">
        ← Back to List
      </router-link>

    <div v-if="loading" class="text-center text-secondary">Loading...</div>
    <div v-else-if="!pokemon" class="text-center text-danger fw-bold">
      Pokémon not found!
    </div>
    <div v-else>
      <h2 class="text-capitalize fw-bold mb-4">{{ pokemon.name }}</h2>
      <div class="d-flex flex-column flex-md-row gap-4">
        <img
            :src="getImage()"
            class="img-fluid rounded shadow-sm"
            style="width: 120px; height: 120px; object-fit: contain; transition: transform 0.4s ease;"
            @mouseover="hovering = true"
            @mouseleave="hovering = false"
            :style="{ transform: hovering ? 'scale(1.1)' : 'scale(1)' }"
        />
        <div class="flex-grow-1">
          <p><strong>Species:</strong> {{ capitalize(pokemon.species.name) }}</p>
          <p><strong>Height:</strong> {{ (pokemon.height / 10).toFixed(1) }} m</p>
          <p>
            <strong>Abilities:</strong>
            {{ pokemon.abilities.map((ab) => ab.ability.name.replace(/-/g, ' ')).join(", ") }}
          </p>
          <h5 class="mt-3">Stats</h5>
          <StatBar v-for="stat in pokemon.stats" :key="stat.stat.name" :stat="stat" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import { useRoute } from "vue-router";
import StatBar from "./StatBar.vue";

const route = useRoute();
const name = route.params.name;

const pokemon = ref(null);
const loading = ref(true);
const hovering = ref(false);

const placeholderImg = "https://via.placeholder.com/96?text=No+Image";

function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

function getImage() {
  if (!pokemon.value) return placeholderImg;

  const animatedSprite =
    pokemon.value.sprites?.versions?.['generation-v']?.['black-white']?.animated?.front_default;

  return (
    animatedSprite ||
    pokemon.value.sprites.front_default ||
    pokemon.value.sprites.front_shiny ||
    placeholderImg
  );
}

async function fetchPokemon() {
  loading.value = true;
  try {
    const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${name}`);
    if (!res.ok) throw new Error("Not found");
    const data = await res.json();
    pokemon.value = data;
  } catch {
    pokemon.value = null;
  }
  loading.value = false;
}

onMounted(fetchPokemon);
</script>
