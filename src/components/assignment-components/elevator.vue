<template>
  <div class="elevator">
    <div class="digital-display">
      <div class="floor-number">{{ currentFloor }}</div>
      <div class="direction-indicator">{{ direction }}</div>
      <div class="status" :class="{ 'emergency': isEmergencyMode }">
        {{ statusMessage }}
      </div>
    </div>

    <div class="elevator-panel">
      <div class="button-grid">
        <button
            v-for="floor in [6, 5, 4, 3, 2, 1]"
            :key="`internal-${floor}`"
            class="floor-button"
            :class="{
            'active': queuedFloors.includes(floor),
            'current': currentFloor === floor
          }"
            :disabled="isButtonDisabled(floor)"
            @click="handleInternalCall(floor)"
        >
          {{ floor }}
        </button>
      </div>

      <div class="control-buttons">
        <button
            class="emergency-button"
            :class="{ 'active': isEmergencyMode }"
            @click="handleEmergencyAction"
        >
          EMERGENCY
        </button>
      </div>
    </div>
  </div>
</template>



<script>
export default {
  name: 'Elevator',

  props: {
    currentFloor: {
      type: Number,
      required: true
    },
    queuedFloors: {
      type: Array,
      default: () => []
    },
    isMoving: {
      type: Boolean,
      default: false
    },
    direction: {
      type: String,
      default: 'IDLE'
    }
  },

  data() {
    return {
      isEmergencyMode: false,
      doorStatus: 'CLOSED',
      statusMessage: 'Ready',
      floorsToVisitInCurrentDirection: [],
      floorsToVisitInOppositeDirection: [],
      currentDirection: 'UP', // or 'DOWN'
    }
  },

  methods: {
    handleInternalCall(floor) {
      this.$emit('internal-call', floor);
    },


    handleExternalCall(floor, direction) {
      if (direction === 'UP') {
        this.moveUp(floor);
      } else if (direction === 'DOWN') {
        this.moveDown(floor);
      }
    },

    processFloors() {
      if (this.isEmergencyMode) {

          this.queuedFloors = []; // clear queue
          this.moveElevator(1);
          return;
        }

      if (this.floorsToVisitInCurrentDirection.length > 0) {
        const nextFloor = this.floorsToVisitInCurrentDirection.shift();
        // Move the elevator to the next floor
        this.moveElevator(nextFloor);
      } else if (this.floorsToVisitInOppositeDirection.length > 0) {
        // Change the direction of the elevator
        this.currentDirection = this.currentDirection === 'UP' ? 'DOWN' : 'UP';
        const nextFloor = this.floorsToVisitInOppositeDirection.shift();
        // Move the elevator to the next floor
        this.moveElevator(nextFloor);
      }
    },
    move() {
      const destinationFloor = this.destinationQueue[0];

      if (this.currentFloor === destinationFloor) {
        this.destinationQueue.shift();
        this.doorStatus = 'OPEN';
        setTimeout(() => {
          this.doorStatus = 'CLOSED';
        }, 3000);
      } else {
        const direction = this.currentFloor < destinationFloor ? 'up' : 'down';
        this.isMoving = true;

        // Move to the next floor with a delay of 3 seconds
        setTimeout(() => {
          this.currentFloor += direction === 'up' ? 1 : -1;
          this.move();
        }, 3000);
      }
    },
    moveElevator(floor) {
      // Calculate the distance to the target floor
      const distance = Math.abs(floor - this.currentFloor);

      // Calculate the time it takes to move the elevator to the target floor
      const time = distance * this.speed; // assuming speed is in floors per second

      // Move the elevator to the target floor
      this.$refs.elevatorCar.style.transform = `translateY(${(floor - this.currentFloor) * this.floorHeight}px)`;

      // Update the current floor
      this.currentFloor = floor;

      // Stop the elevator at the target floor
      this.stopElevator();

      // Open the doors
      this.openDoors();

      // After some time, close the doors and proceed to the next floor
      setTimeout(() => {
        this.closeDoors();
        // Check if there are more floors in the queue
        if (this.queuedFloors.length > 0) {
          const nextFloor = this.queuedFloors.shift();
          this.moveElevator(nextFloor);
        }
      }, 3000);
    },
    handleEmergencyAction() {
      this.isEmergencyMode = !this.isEmergencyMode;
      this.statusMessage = this.isEmergencyMode ? 'EMERGENCY MODE' : 'Ready';
      this.$emit('emergency-toggle', this.isEmergencyMode);
    },

    handleDoorAction() {
      this.doorStatus = this.doorStatus === 'CLOSED' ? 'OPEN' : 'CLOSED';
      this.$emit('door-toggle', this.doorStatus);
    },

    updateState(state) {
      Object.assign(this.$props, state);
    },

    isButtonDisabled(floor) {
      return this.queuedFloors.includes(floor) ||
          this.currentFloor === floor ||
          this.isEmergencyMode;
    }
  }
}
</script>


<style scoped>
.elevator-container {
  width: 300px;
  padding: 20px;
  background: #2c3e50;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.elevator-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.digital-display {
  background: #000;
  padding: 15px;
  border-radius: 8px;
  text-align: center;
  color: #00ff00;
  font-family: 'Digital', monospace;
}

.floor-number {
  font-size: 48px;
  font-weight: bold;
}

.direction {
  font-size: 24px;
  margin: 5px 0;
}

.status {
  font-size: 14px;
}

.status.emergency {
  color: #ff4444;
  animation: blink 1s infinite;
}

.button-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.floor-buttons {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
  justify-items: center;
}
.elevator {
  background: #2c3e50;
  border-radius: 12px;
  padding: 20px;
  width: 300px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
}

.digital-display {
  background: #000;
  color: #00ff00;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  text-align: center;
  font-family: 'Digital', monospace;
  box-shadow: inset 0 0 10px rgba(0, 255, 0, 0.3);
}

.floor-number {
  font-size: 36px;
  font-weight: bold;
  margin-bottom: 5px;
}

.direction-indicator {
  font-size: 18px;
  color: #39ff14;
}

.status {
  font-size: 14px;
  color: #39ff14;
  margin-top: 5px;
}

.status.emergency {
  color: #ff4444;
  animation: blink 1s infinite;
}

.elevator-panel {
  background: #34495e;
  padding: 20px;
  border-radius: 8px;
}

.button-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
  margin-bottom: 20px;
}

.floor-button {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: none;
  background: linear-gradient(145deg, #2c3e50, #34495e);
  color: #fff;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 4px 4px 8px #1a252f,
  -4px -4px 8px #3e5871;
}

.floor-button:hover:not(:disabled) {
  transform: scale(1.05);
  box-shadow: 6px 6px 12px #1a252f,
  -6px -6px 12px #3e5871;
}

.floor-button:active:not(:disabled) {
  transform: scale(0.95);
  box-shadow: inset 4px 4px 8px #1a252f,
  inset -4px -4px 8px #3e5871;
}

.floor-button.active {
  background: #3498db;
  box-shadow: inset 4px 4px 8px #2980b9,
  inset -4px -4px 8px #40a9ed;
}

.floor-button.current {
  background: #2ecc71;
  box-shadow: inset 4px 4px 8px #27ae60,
  inset -4px -4px 8px #35ea82;
}

.floor-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  background: #95a5a6;
}

.control-buttons {
  display: flex;
  gap: 15px;
  margin-top: 20px;
}

.emergency-button, .door-button {
  flex: 1;
  padding: 15px;
  border: none;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  text-transform: uppercase;
  font-size: 14px;
}

.emergency-button {
  background: #e74c3c;
  color: white;
  box-shadow: 4px 4px 8px #c0392b,
  -4px -4px 8px #ff5447;
}

.emergency-button.active {
  background: #c0392b;
  animation: pulse 1s infinite;
  box-shadow: inset 4px 4px 8px #962d22,
  inset -4px -4px 8px #ea4534;
}

.door-button {
  background: #27ae60;
  color: white;
  box-shadow: 4px 4px 8px #219a52,
  -4px -4px 8px #2dc26e;
}

.door-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  box-shadow: none;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes blink {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
  100% {
    opacity: 1;
  }
}

/* Add metallic texture */
.elevator::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg,
  rgba(255, 255, 255, 0.1) 0%,
  rgba(255, 255, 255, 0) 100%);
  pointer-events: none;
  border-radius: 12px;
}

/* Responsive adjustments */
@media (max-width: 400px) {
  .elevator {
    width: 100%;
    padding: 15px;
  }

  .button-grid {
    gap: 10px;
  }

  .floor-button {
    width: 50px;
    height: 50px;
    font-size: 18px;
  }
}
.floor-button {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: none;
  background: #34495e;
  color: white;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
}

.floor-button:hover:not(:disabled) {
  background: #3498db;
}

.floor-button.active {
  background: #27ae60;
}

.floor-button.queued {
  background: #f1c40f;
  animation: pulse 1s infinite;
}

.floor-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.control-section {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.emergency-button, .door-control {
  flex: 1;
  padding: 15px;
  border: none;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
  text-transform: uppercase;
}

.emergency-button {
  background: #e74c3c;
  color: white;
}

.emergency-button.active {
  background: #c0392b;
  animation: pulse 1s infinite;
}

.door-control {
  background: #27ae60;
  color: white;
}

.door-control:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

@keyframes blink {
  0% { opacity: 1; }
  50% { opacity: 0; }
  100% { opacity: 1; }
}
</style>
