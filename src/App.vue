<script setup>
</script>

<template>
  <div class="container py-4">
    <h1 class="text-center mb-4">Pokémon Mando</h1>
    <h2 class="text-center mb-4">Pokémon Search</h2>
    <transition name="fade" mode="out-in">
      <router-view />
    </transition>
  </div>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
