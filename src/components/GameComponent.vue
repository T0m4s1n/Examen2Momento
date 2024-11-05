<script lang="ts">
import { defineComponent, ref, computed } from 'vue';
  
interface Location {
  id: number;
  name: string;
  description: string;
}

class Position {
  data: Location;
  next: Position | null;
  prev: Position | null;

  constructor(data: Location) {
    this.data = data;
    this.next = null;
    this.prev = null;
  }
}

class GameMap {
  head: Position | null;
  tail: Position | null;
  current: Position | null;
  isCircular: boolean;
  isDouble: boolean;

  constructor(isCircular = false, isDouble = false) {
    this.head = null;
    this.tail = null;
    this.current = null;
    this.isCircular = isCircular;
    this.isDouble = isDouble;
  }

  addLocation(location: Location): void {
    const newPosition = new Position(location);

    if (!this.head) {
      this.head = newPosition;
      this.tail = newPosition;
      this.current = newPosition;
    } else {
      if (this.tail) {
        this.tail.next = newPosition;
        if (this.isDouble) {
          newPosition.prev = this.tail;
        }
        this.tail = newPosition;

        if (this.isCircular) {
          this.tail.next = this.head;
          if (this.isDouble) {
            this.head.prev = this.tail;
          }
        }
      }
    }
  }

  moveForward(): boolean {
    if (!this.current) return false;
    
    if (this.current.next) {
      this.current = this.current.next;
      return true;
    }
    return false;
  }

  moveBackward(): boolean {
    if (!this.current || !this.isDouble) return false;
    
    if (this.current.prev) {
      this.current = this.current.prev;
      return true;
    }
    return false;
  }
}

export default defineComponent({
  name: 'GameComponent',
  setup() {
    const gameMap = ref<GameMap | null>(null);
    const selectedMapType = ref('simple');
    const currentNode = ref<Position | null>(null);

    const locations: Location[] = [
      { id: 1, name: 'T0m4s1n\'s Lair', description: 'A cave in a mountain where the programmer T0m4s1n codes his strange things.' },
      { id: 2, name: 'Obonuco City', description: 'A small town under a rock' },
      { id: 3, name: 'The COOPERATIVE UNIVERSITY OF COLOMBIA', description: 'A place of fear and perdition where students fight for grades' },
      { id: 4, name: 'Dark souls?', description: 'A place which you will never return from' },
      { id: 5, name: 'idk', description: 'IDK teacher i ran out of ideas' }
    ];

    const initializeMap = () => {
      const isCircular = ['circular', 'doubleCircular'].includes(selectedMapType.value);
      const isDouble = ['double', 'doubleCircular'].includes(selectedMapType.value);
      
      gameMap.value = new GameMap(isCircular, isDouble);
      
      locations.forEach(location => {
        gameMap.value?.addLocation(location);
      });
      
      currentNode.value = gameMap.value?.current || null;
    };

    const moveForward = () => {
      if (gameMap.value?.moveForward()) {
        currentNode.value = gameMap.value.current;
      }
    };

    const moveBackward = () => {
      if (gameMap.value?.moveBackward()) {
        currentNode.value = gameMap.value.current;
      }
    };

    const canMoveForward = computed(() => {
      if (!gameMap.value?.current) return false;
      return gameMap.value.current.next !== null;
    });

    const canMoveBackward = computed(() => {
      if (!gameMap.value?.current || !gameMap.value.isDouble) return false;
      return gameMap.value.current.prev !== null;
    });
    initializeMap();

    return {
      currentNode,
      selectedMapType,
      moveForward,
      moveBackward,
      initializeMap,
      canMoveForward,
      canMoveBackward
    };
  }
});
</script>
<template>
    <div class="game-map">
      <div class="controls">
        <button @click="moveForward" :disabled="!canMoveForward">Next</button>
        <button @click="moveBackward" :disabled="!canMoveBackward">Before</button>
      </div>
      
      <div class="current-location">
        <h2>Actual Location</h2>
        <p>{{ currentNode?.data.name }}</p>
        <p>{{ currentNode?.data.description }}</p>
      </div>
      
      <div class="map-type">
        <select v-model="selectedMapType" @change="initializeMap">
          <option value="simple">Simple List</option>
          <option value="double">Double List</option>
          <option value="circular">Circular List</option>
          <option value="doubleCircular">Double Circular List</option>
        </select>
      </div>
    </div>
    </template>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;1,100;1,200;1,300;1,400;1,500;1,600;1,700&family=Lato:ital,wght@0,100;0,300;0,400;0,700;0,900;1,100;1,300;1,400;1,700;1,900&family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&family=Quicksand:wght@300..700&display=swap');

.current-location {
  background: linear-gradient(45deg, #1a1a1a, #2a2a2a, #1a1a1a);
  border: 1px solid #e9ecef;
  border-radius: 0.25rem;
  padding: 1rem;
  margin-bottom: 1rem;
  font-family: "Poppins", sans-serif;
  width: 15rem;
  height: 20rem;
}

  .game-map {
    padding: 20px;
    max-width: 600px;
    margin: 0 auto;
  }
  
  .controls {
    margin-bottom: 20px;
  }
  
    button {
    padding: 0.75rem 1.75rem;
    border: 2px solid rgba(255, 255, 255, 0.1);
    border-radius: 0.5rem;
    background: linear-gradient(45deg, #1a1a1a, #2a2a2a, #1a1a1a);
    background-size: 200% 100%;
    color: white;
    font-weight: 500;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    transition: all 0.3s ease;
    min-width: 120px;
    justify-content: space-between;
    font-family: "Poppins", sans-serif;
}

button:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
    border-color: rgba(255, 255, 255, 0.2);
}

button-btn[data-effect="shine"]::before {
    content: '';
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: rgba(255, 255, 255, 0.1);
    transform: rotate(45deg);
    animation: shine 3s infinite;
}
  
  button:disabled {
    cursor: not-allowed;
    opacity: 0.5;
  }  
  .map-type {
    margin-top: 20px;
  }
  
  select {
  padding: 8px;
  width: 200px;
  font-size: 16px;
  font-family: "Poppins", sans-serif;
  border-radius: 0.5rem;
  background: linear-gradient(45deg, #1a1a1a, #2a2a2a, #1a1a1a);
  background-size: 200% 100%;
  color: white;
  font-weight: 500;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
  min-width: 120px;
  justify-content: space-between;
  font-family: "Poppins", sans-serif;
}

select option:checked {
  background-color: #3e8e41;
  color: white;
  font-weight: bold;
}

select option {
  background-color: #2a2a2a;
  color: #ccc;
}

select option:hover {
  background-color: #3e8e41;
  color: white;
}

  select:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
    border-color: rgba(255, 255, 255, 0.2);
  }

    select:disabled {
        cursor: not-allowed;
        opacity: 0.5;
    }

    select[data-effect="shine"]::before {
        content: '';
        position: absolute;
        top: -50%;
        left: -50%;
        width: 200%;
        height: 200%;
        background: rgba(255, 255, 255, 0.1);
        transform: rotate(45deg);
        animation: shine 3s infinite;
    }

  @keyframes shine {
    0% {
        opacity: 0;
        transform: rotate(45deg) translateX(-100%);
    }
    15% {
        opacity: 1;
    }
    35% {
        opacity: 1;
    }
    100% {
        opacity: 0;
        transform: rotate(45deg) translateX(100%);
    }
}

@keyframes text {
    0% {
        background-position: 100% center;
    }    
    100% {
        background-position: -100% center;
    }    
}

  </style>
