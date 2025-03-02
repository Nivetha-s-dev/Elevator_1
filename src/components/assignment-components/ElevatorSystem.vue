<!-- src/components/ElevatorSystem.vue -->
<template>
  <div class="elevator-system">
    <ElevatorBank
        ref="elevatorBank"
        @floor-call="handleSystemFloorCall"
    />
  </div>
</template>

<script>
import ElevatorBank from './elevator-bank.vue'

export default {
  name: 'ElevatorSystem',
  components: {
    ElevatorBank
  },

  data() {
    return {
      elevatorState: {
        currentFloor: 1,
        targetFloor: null,
        queuedFloors: [],
        isMoving: false,
        direction: 'IDLE'
      }
    }
  },

  methods: {
    handleSystemFloorCall(floor) {
      // Update the shared state
      if (!this.elevatorState.queuedFloors.includes(floor)) {
        this.elevatorState.queuedFloors.push(floor);
        // Notify elevator bank of the change
        this.$refs.elevatorBank.updateElevatorState(this.elevatorState);
      }
    }
  }
}
</script>


<style scoped>
.elevator-system {
  position: relative;
  height: 600px;
  display: flex;
  justify-content: space-between;
  padding: 20px;
  background-color: #f5f5f5;
}

.elevator {
  position: absolute;
  left: 100px;
  width: 100px;
  height: 120px;
  background-color: #ddd;
  border: 2px solid #999;
  transition: bottom 0.5s linear;
}

.doors {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.door {
  position: absolute;
  width: 50%;
  height: 100%;
  background-color: #666;
  transition: transform 3s linear;
}

.door.left {
  left: 0;
  border-right: 1px solid #444;
}

.door.right {
  right: 0;
  border-left: 1px solid #444;
}

.doors-open .door.left {
  transform: translateX(-100%);
}

.doors-open .door.right {
  transform: translateX(100%);
}

.floor-panel {
  position: absolute;
  right: 50px;
  display: flex;
  flex-direction: column-reverse;
  gap: 30px;
}

.floor-buttons {
  display: flex;
  align-items: center;
  gap: 10px;
}

.floor-number {
  font-weight: bold;
  min-width: 80px;
}

.elevator-panel {
  position: absolute;
  left: 250px;
  background-color: #333;
  padding: 15px;
  border-radius: 8px;
  color: white;
}

.internal-buttons {
  display: flex;
  flex-direction: column-reverse;
  gap: 10px;
}

button {
  padding: 10px 15px;
  cursor: pointer;
  border: none;
  border-radius: 4px;
  background-color: #4CAF50;
  color: white;
  font-weight: bold;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  background-color: #999;
}

.floor-display {
  position: absolute;
  top: -25px;
  width: 100%;
  text-align: center;
  background-color: #000;
  color: #00ff00;
  padding: 5px;
  font-family: 'Digital', sans-serif;
  font-size: 1.2em;
}

.call-button {
  background-color: #2196F3;
}

.internal-button {
  background-color: #FFA000;
  width: 50px;
  height: 50px;
  border-radius: 50%;
}

h3 {
  margin: 0 0 15px 0;
  text-align: center;
}
</style>
