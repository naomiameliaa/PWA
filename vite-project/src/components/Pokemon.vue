<template>
    <div>
      <h1>{{ title }}</h1>
      <div v-for="item in items">
        <a @click="getImage(item.name)">{{ item.name }}</a>
      </div>
    </div>
  </template>
  
  <script setup>
    import { ref, onMounted } from 'vue';

    // Define reactive variables
    const title = ref('Fetching Data Example');
    const items = ref([]);

    async function getImage(name){
      try {
        const response = await fetch('https://pokeapi.co/api/v2/pokemon/'+name);
        if(response.ok){
          const data = await response.json();
          return window.open(data.sprites.back_default) 
        } else {
          console.error('Failed to fetch image data');
        }
      } catch (error) {
        console.error('An error occurred:', error);
      }
    }
    
    // Define a function to fetch data
    const fetchData = async () => {
      try {
        const response = await fetch('https://pokeapi.co/api/v2/pokemon');
        if (response.ok) {
          const data = await response.json();
          items.value = data.results;
        } else {
          console.error('Failed to fetch data');
        }
      } catch (error) {
        console.error('An error occurred:', error);
      }
    };

    onMounted(fetchData);

  </script>