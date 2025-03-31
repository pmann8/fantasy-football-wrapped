<script setup lang="ts">
import { onMounted, ref, watch, computed } from "vue";
import { useStore } from "../../store/store";
const store = useStore();
const systemDarkMode = window.matchMedia(
  "(prefers-color-scheme: dark)"
).matches;
const clicked = ref(systemDarkMode);

onMounted(() => {
  if (systemDarkMode && !localStorage.darkMode) {
    clicked.value = true;
    store.updateDarkMode(true);
  } else if (localStorage.darkMode) {
    clicked.value = JSON.parse(localStorage.darkMode);
    store.updateDarkMode(clicked.value);
  }
});

watch(clicked, () => {
  localStorage.darkMode = clicked.value;
  store.updateDarkMode(clicked.value);
});

const darkMode = computed(() => {
  return store.darkMode;
});

const setColorMode = () => {
  clicked.value = !clicked.value;
};
</script>
<template>
    <div class="relative pt-5 text-center">
        <h1 class="px-1 font-bold tracking-tight text-center text-gray-900 custom-font-size sm:leading-none lg:px-24 xl:px-56 md:text-5xl lg:text-6xl dark:text-white">
            <span class="text-transparent bg-clip-text bg-gradient-to-r to-indigo-600 from-sky-500 dark:to-indigo-500 dark:from-sky-400">
                Soup Hat Dynasty
            </span>
        </h1>
        <h2 class="max-w-4xl mx-auto mt-5 mb-0 text-lg font-normal text-center text-gray-600 text-balance lg:text-xl sm:px-16 dark:text-gray-300">
            A detailed analysis and elegant charts for the Soup Hat Dynasty League.
        </h2>
    </div>
</template>
<style scoped>
.custom-font {
  font-family: "Josefin Sans", sans-serif;
  font-optical-sizing: auto;
  font-weight: 600;
  font-style: normal;
  font-size: 1.6rem;
}
</style>
