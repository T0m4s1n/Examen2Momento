<script setup lang="ts">
import GameComponent from '@/components/GameComponent.vue';
defineProps<{
  name: string;
}>();
</script>
<template>
  <main>
    <GameComponent />
  </main>
</template>
