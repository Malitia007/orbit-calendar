<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';
import OrbContent from "@/components/OrbContent.vue";

const props = defineProps({
  baseRadius: { type: Number, default: 245 },
  radiusIncrement: {type: Number, default: 75},
  numDaysVisible: {type: Number, default: 10}
});

const orbits = ref([]);
const startDate = ref(new Date());
const visibleOrbits = ref([]);
const currentSet = ref(0);
const activeOrbit= ref(new Array(9).fill(false));
const isNext = ref(false);
const isPrev = ref(false);
const orbitScale = ref(0);

const baseRadius =computed(()=>{
  if (orbitScale.value === 0) return props.baseRadius
  if (orbitScale.value === -1) return props.baseRadius - props.radiusIncrement
  if (orbitScale.value === 1) return props.baseRadius + props.radiusIncrement
})


const fetchOrbits = async (start_date) => {
  try {
    const response = await axios.get(`https://xsrr-l2ye-dpbj.f2.xano.io/api:oUvfVMO5/receive_week`, {
      params: {
        start_date: start_date
      }
    });
    orbits.value = response.data;
    visibleOrbits.value = orbits.value.slice(0, props.numDaysVisible);
  } catch (error) {
    console.error("Error fetching data:", error);
  }
};

const handleScroll = (event) => {
  if (event.deltaY > 0) {
    isNext.value = true;
    // visibleOrbits.value.push({array:[]})
    orbitScale.value = 1;
    setTimeout(()=>{
      isNext.value = false;
      orbitScale.value = 0;
      nextSet();
      }, 500)
  } else {
    isPrev.value = true;
    // visibleOrbits.value.unshift({array:[]})
    orbitScale.value = -1;
    setTimeout(()=>{
      isPrev.value = false;
      orbitScale.value = 0;
      prevSet();
      }, 500)
  }
};

const nextSet = () => {
  if ((currentSet.value + 1) * props.numDaysVisible < orbits.value.length) {
    currentSet.value++;
    updateVisibleOrbits();
  } else {
    const newStartDate = new Date(startDate.value);
    newStartDate.setDate(startDate.value.getDate() + props.numDaysVisible);
    startDate.value = newStartDate;
    fetchOrbits(newStartDate);
  }
};

const prevSet = () => {
  if (currentSet.value > 0) {
    currentSet.value--;
    updateVisibleOrbits();
  } else {
    const newStartDate = new Date(startDate.value);
    newStartDate.setDate(startDate.value.getDate() - props.numDaysVisible);
    startDate.value = newStartDate;
    fetchOrbits(newStartDate);
  }
};

const updateVisibleOrbits = () => {
  const startIndex = currentSet.value * props.numDaysVisible;
  visibleOrbits.value = orbits.value.slice(startIndex, startIndex + props.numDaysVisible);
};

const orbitStyle = (dayIndex) => {
  const radius = baseRadius.value + props.radiusIncrement * dayIndex
  return {
    width:  `${radius * 2}px`,
    height: `${radius * 2}px`,
    bottom: `${-radius}px`,
    zIndex: `${props.numDaysVisible*2-dayIndex}`
  };
};

const debounce = (func, delay) => {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), delay);
  };
};

const debouncedHandleScroll = debounce(handleScroll, 500);

onMounted(() => {
  fetchOrbits(startDate.value);
});

</script>

<template>
  <div class="orbit-nav" @wheel="debouncedHandleScroll " :class="[]">
    <div
        class="orbit"
        :class="[{'active-orbit':activeOrbit[dayIndex]}]"
        v-for="(orbit, dayIndex) in visibleOrbits"
        :key="dayIndex"
        :style="orbitStyle(dayIndex)"
    >
      <orb-content
          @orbHovered="activeOrbit[dayIndex]=true" @orbLeft="activeOrbit[dayIndex]=false"
          :orbit-infos="orbit.array"
          :radius-increment="props.radiusIncrement"
          :base-radius="baseRadius"
          :dayIndex="dayIndex"/>
    </div>
    <div class="date-display">
      {{ startDate }}
    </div>
  </div>
</template>

<style scoped>

.orbit-nav {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100%;
  position: relative;
  background: black;
  overflow: hidden;
}

.orbit {
  position: absolute;
  border-radius: 50%;
  border: 3px solid #eee;
  display: flex;
  justify-content: center;
  align-items: center;
  left: 0;
  right: 0;
  margin: auto;
  opacity: 1;
  transition: all 0.5s linear;
}

.prev-orbit {
   transition: all 0.5s linear;
  //opacity: 0;
}

.next-orbit {
  transition: all 0.5s linear;
}

.orbit.active-orbit {
  z-index: 9999!important;
}

.orbit:before {
  content: "";
  display: block;
  position: absolute;
  width: calc(100% + 4px);
  height: 100%;
  border: 2px solid #000;
  border-radius: inherit;
  left: -2px;
  bottom: 0;
}

.orbit.active {
  opacity: 1;
}

.date-display {
  position: absolute;
  top: 10%;
  font-size: 24px;
  color: #333;
}

</style>