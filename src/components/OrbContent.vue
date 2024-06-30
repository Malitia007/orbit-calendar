<script setup>
import { ref, onMounted } from 'vue';
const props = defineProps({
  orbitInfos: {type: Array, default: []},
  baseRadius: {type: Number, default: 245},
  radiusIncrement: {type: Number, default: 75},
  dayIndex: {type: Number, default: 1}
})

defineEmits(['orbHovered', 'orbLeft']);

const orbStyle = (idx, length, dayIndex) => {
  const startRadians = 180*Math.PI/180 + 180*Math.PI/180/(length===1?2:length===2?1.5:length);
  const endRadians   = 360*Math.PI/180 - 180*Math.PI/180/(length===1?2:length===2?1.5:length);
  const stepRadians = (endRadians - startRadians) / (length===1?2:length-1);
  const angle =  idx%length*stepRadians+startRadians;
  const semicircleRadius = props.baseRadius + (props.radiusIncrement * dayIndex);
  const x = semicircleRadius + semicircleRadius * Math.cos(angle);
  const y = semicircleRadius + semicircleRadius * Math.sin(angle);
  return {
    left: `${x}px`,
    top: `${y}px`,
    transform: `translate(-50%, -50%)`, // Center the orb
  };
};

const  formatDate = (date) => {
  const options = {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: 'numeric',
    minute: 'numeric',
    hour12: true,
    timeZoneName: 'short'
  };

  return date.toLocaleString('en-US', options);
}

const isTruncated = ref(true);

</script>

<template>
  <div
      @mouseenter="$emit('orbHovered')"
      @mouseleave="$emit('orbLeft'); isTruncated=true"
      class="orb"
      v-for="(orb, idx) in props.orbitInfos"
      :key="idx"
      :style="orbStyle(idx, props.orbitInfos.length, props.dayIndex)"
  >
    <div class="orb-image" :style="{background: `url(${orb?.img})`}"></div>
    <div class="info-card">
      <div class="info-person">
        <div class="img-container" :style="{backgroundImage: `url(${orb?.img})`}"></div>
        <div class="shadow-border"></div>
        <div class="flex-auto">
          <div class="d-flex justify-space-around">
            <div class="flex-auto">
              <h4>{{orb?.name}}</h4>
              <p>{{orb?.position}}</p>
              <p>{{orb?.city}}</p>
            </div>
            <div>
              <img src="/src/assets/company-avatar.svg" alt="logo"/>
            </div>
        </div>
          <div class="mutual">
            <div><img src="/src/assets/avatar-group.png" alt="users group" /></div>
            <p>Jason Diamond, John Eremic, <br/> and 281 other mutual connections</p>
          </div>
        </div>
      </div>

      <div class="replay">
        <a href=""><img src="../assets/Vector.svg"/>
          Reply from Emery Wells</a>
      </div>

      <div class="msg-card">
        <div class="msg-date">{{formatDate(new Date(orb?.created_at))}}</div>
        <p><strong>{{orb?._orbits_last_message?.message_head}}</strong></p>
        <p :class="{ 'text-truncated': isTruncated }">{{orb?._orbits_last_message?.message}}</p>
        <button v-if="isTruncated" @click="isTruncated = !isTruncated">More <img src="../assets/chevron-down.svg" alt="chevron"/> </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.orb {
  position: absolute;
  width: 60px;
  height: 60px;
  margin: 0;
  border-radius: 50%;
  background-color: #333;
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
  transition: left 0.5s linear, top 0.5s linear ;
}

.orb-image {
  position: absolute;
  overflow: hidden;
  width: 100%;
  height: 100%;
  background-size: cover!important;
  border-radius: 50%;
  -webkit-box-shadow: 0px 0px 0px 4px rgba(0,0,0,1);
  -moz-box-shadow: 0px 0px 0px 4px rgba(0,0,0, 1);
  box-shadow: 0px 0px 0px 4px rgba(0,0,0,1);
}

.orb-image:after {
  content: "";
  display: block;
  width: calc(100% + 3px);
  height: calc(100% + 3px);
  border: 3px solid #ffffff;
  border-radius: 50%;
  position: absolute;
  top: 0px;
  left: -1px;
}

.info-card {
  position: absolute;
  top: -28px;
  left: -28px;
  display: none;
  width: 502px;
  border-radius: 10px;
  background: linear-gradient(#0c0c0c, #0c0c0c) padding-box,
  linear-gradient(to bottom, white, #222) border-box;
  border: 1px solid transparent;
  color: #ffffff;
  padding: 8px;
  .replay {
    background: #161616;
    padding: 10px 8px;
    margin: 0 -8px;
    display: flex;
    justify-items: center;
    vertical-align: middle;
    a{
      color: white;
      text-decoration: none;
      img{
        position: relative;
        top: 3px;
        margin-right: 3px;
      }
    }
  }
  .msg-card {
    font-size: 1rem;
    .msg-date {
      font-size: 0.875rem;
      color: #929292;
      padding-top: 0.5em;
    }
    button {
      background: transparent;
      color: white;
      border: none;
      cursor: pointer;
      font-weight: 300;
    }
    img{
      vertical-align: middle;
    }
  }
}

.img-container {
  position: relative;
  width: 100px;
  height: 100px;
  margin-right: 10px;
  border-radius: 50%;
  background-size: cover;
  background-position: center;
  z-index: 1;
}

.shadow-border {
  content: "";
  position: absolute;
  display: block;
  width: 102px;
  height: 102px;
  background: linear-gradient(transparent, transparent) padding-box,
  linear-gradient(to bottom, white, #222) border-box;
  border: 2px solid transparent;
  top: 7px;
  left: 7px;
  border-radius: 50%;
  z-index: 0;
}

.img-container:hover + .shadow-border{
  width: 106px;
  height: 106px;
  border: 4px solid transparent;
  top: 5px;
  left: 5px;
}

.info-person{
  display: flex;
  h4{
    font-size: 1rem;
    font-weight: bold;
  }
  p{
    font-size: 0.75rem;
    line-height: 1.5em;
  }
  .mutual {
    display: flex;
    flex-wrap: wrap;
    div {
      margin-right: 8px;
      padding-top: 0.2em;
    }
    p {
      margin: 0;
      line-height: 1.2em;
    }
  }
}

.text-truncated {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}


.orb:hover {
  z-index: 9999;
}

.orb:hover .info-card{
  display: block;
}
</style>