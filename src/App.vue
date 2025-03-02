<!-- App.vue -->
<template>
  <div id="app">
    <div class="elevator-system">
      <!-- Floors Section -->
      <div class="floors-section">
        <Floor
            v-for="floor in [...floors].reverse()"
            :key="floor"
            :floor="floor"
            :current-floor="currentFloor"
            :door-status="doorStatus"
            :is-moving="isMoving"
            :is-emergency-mode="isEmergencyMode"
            @floor-call="requestFloor"
            @emergency-toggle="handleEmergencyToggle"
        />
      </div>

      <!-- Elevator Section -->
      <div class="elevator-section">
        <elevator
            :current-floor="currentFloor"
            :queued-floors="queuedFloors"
            :is-moving="isMoving"
            :direction="direction"
            :is-emergency-mode="isEmergencyMode"
            @internal-call="handleInternalCall"
            @emergency-toggle="handleEmergencyToggle"
        />
      </div>
    </div>
  </div>
</template>

<script>
import Elevator from './components/assignment-components/elevator.vue'
import Floor from './components/assignment-components/floor.vue'

export default {
  name: 'App',
  components: {
    Elevator,
    Floor
  },
  data() {
    return {
      floors: [1],
      currentFloor: 1,
      queuedFloors: [],
      isMoving: false,
      direction: 'IDLE',
      isEmergencyMode: false,
      doorStatus: 'CLOSED',
      disableInputs: false
    }
  },
  methods: {

    handleFloorCall(floor) {
      this.$refs.elevator.handleExternalCall(floor);
    },
    handleInternalCall(floor) {
      this.requestFloor(floor);
    },
    requestFloor(floor) {
      if (!this.isEmergencyMode && !this.disableInputs && !this.queuedFloors.includes(floor) && floor !== this.currentFloor) {
        this.queuedFloors.push(floor);
        this.processQueue();
      }
    },
    handleEmergencyToggle(status) {
      this.isEmergencyMode = status;
      if (status) {
        // immediately stop movement and go to floor 1
        this.isMoving = false;
        this.direction = 'IDLE';
        this.doorStatus = 'OPEN';
        this.currentFloor = 1;
        this.queuedFloors = []; // clear queue
        this.processQueue(); // restart queue processing
      } else {
        this.doorStatus = 'CLOSED';
      }
    },
    clearQueue() {
      this.queuedFloors = [];
    },
    async processQueue() {
      if (this.isMoving || this.isEmergencyMode || this.queuedFloors.length === 0) {
        return;
      }
      this.isMoving = true;
      this.doorStatus = 'CLOSED';

      while (this.queuedFloors.length > 0 && !this.isEmergencyMode) {
        const targetFloor = this.queuedFloors[0];
        this.direction = targetFloor > this.currentFloor ? '▲' : '▼';

        while (this.currentFloor !== targetFloor && !this.isEmergencyMode) {
          await new Promise(resolve => setTimeout(resolve, 3000));
          this.currentFloor += targetFloor > this.currentFloor ? 1 : -1;
        }

        if (!this.isEmergencyMode) {
          this.queuedFloors.shift();
          this.doorStatus = 'OPEN';
          await new Promise(resolve => setTimeout(resolve, 2000));
          if (!this.isEmergencyMode) {
            this.doorStatus = 'CLOSED';
          }
        }
      }

      this.isMoving = false;
      this.direction = 'IDLE';
    }
  }
}
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  padding: 20px;
}

.elevator-system {
  display: flex;
  gap: 40px;
  max-width: 1200px;
  margin: 0 auto;
}

.floors-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.elevator-section {
  flex: 1;
}
</style>
