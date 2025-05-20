<template>
    <div class="d-flex justify-content-end mb-3">
        <button class="btn btn-sm btn-outline-secondary" @click="toggleTheme">
            {{ isDarkMode ? "Light Mode" : "Dark Mode" }}
        </button>
    </div>
  <div class="['card p-4 shadow-sm', { 'bg-dark text-white': isDarkMode }]">
    <input
      v-model="search"
      type="text"
      class="form-control mb-3"
      placeholder="Search Pokémon by name..."
      @input="onSearchChange"
    />

    <div v-if="loading" class="text-center text-secondary">
      Loading...
    </div>

    <div v-else>
      <ul class="list-group mb-3" v-if="pokemonList.length">
        <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
           <div
                v-for="p in pokemonList"
                :key="p.name"
                class="col animate__animated animate__fadeIn"
            >
                <div class="['card h-100 shadow-sm border-0 text-center p-3', { 'bg-dark text-white': isDarkMode }]">
                    <img
                        :src="pokemonImages[p.name] || placeholderImg"
                        class="mx-auto mb-2"
                        style="width: 80px; height: 80px; object-fit: contain"
                        @error="onImageError"
                    />
                    <h5 class="text-capitalize">{{ p.name }}</h5>
                    <router-link
                        :to="`/pokemon/${p.name}`"
                        class="btn btn-outline-warning btn-sm mt-2"
                    >
                    Detail
                    </router-link>
                </div>
            </div>
        </div>
      </ul>

      <div v-else class="['text-center', isDarkMode ? 'text-light' : 'text-danger']">No Pokémon found.</div>

      <nav v-if="!search" aria-label="Page navigation example">
        <ul class="pagination justify-content-center">
          <li :class="['page-item', { disabled: page === 1 }]">
            <button class="page-link" @click="prevPage">Prev</button>
          </li>
          <li class="page-item disabled">
            <span class="page-link">Page {{ page }}</span>
          </li>
          <li class="page-item">
            <button class="page-link" @click="nextPage">Next</button>
          </li>
        </ul>
      </nav>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, watchEffect, onMounted, watch } from "vue";

const LIMIT = 21;
const search = ref("");
const page = ref(1);
const pokemonList = ref([]);
const loading = ref(false);

const pokemonImages = reactive({});
const placeholderImg = "https://via.placeholder.com/96?text=No+Image";

const isDarkMode = ref(false);

watch(isDarkMode, (val) => {
  document.body.classList.toggle("bg-dark", val);
  document.body.classList.toggle("text-white", val);
});

onMounted(() => {
  isDarkMode.value = localStorage.getItem("theme") === "dark";
});

function toggleTheme() {
  isDarkMode.value = !isDarkMode.value;
  localStorage.setItem("theme", isDarkMode.value ? "dark" : "light");
}

async function getPokemonImage(name) {
  if (pokemonImages[name]) return pokemonImages[name];

  try {
    const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${name}`);
    const data = await res.json();
    const sprite =
      data.sprites?.versions?.['generation-v']?.['black-white']?.animated?.front_default ||
      data.sprites.front_default ||
      data.sprites.front_shiny;
    pokemonImages[name] = sprite || placeholderImg;
    return pokemonImages[name];
  } catch {
    pokemonImages[name] = placeholderImg;
    return placeholderImg;
  }
}

async function fetchList() {
  loading.value = true;
  const offset = (page.value - 1) * LIMIT;

  if (search.value.trim()) {
    try {
      const res = await fetch(
        `https://pokeapi.co/api/v2/pokemon/${search.value.toLowerCase()}`
      );
      if (!res.ok) throw new Error("Not found");
      const data = await res.json();
      pokemonList.value = [data];

      await getPokemonImage(data.name);
    } catch {
      pokemonList.value = [];
    }
    loading.value = false;
    return;
  }

  try {
    const res = await fetch(
      `https://pokeapi.co/api/v2/pokemon?limit=${LIMIT}&offset=${offset}`
    );
    const data = await res.json();
    pokemonList.value = data.results;

    for (const p of data.results) {
      await getPokemonImage(p.name);
    }
  } catch {
    pokemonList.value = [];
  }

  loading.value = false;
}

function prevPage() {
  if (page.value > 1) page.value--;
}

function nextPage() {
  page.value++;
}

function onSearchChange() {
  page.value = 1;
}

function onImageError(event) {
  event.target.src = placeholderImg;
}

watchEffect(() => {
  fetchList();
});
</script>