<template>
  <div
    class="userLayout flex-auto flex flex-col"
    :class="box ? 'h-screen perspective' : ''"
  >
    <router-view v-slot="{ Component, route }">
      <transition :name="box ? route.meta.transition : ''">
        <keep-alive>
          <div
            :key="route.meta.transition"
            class="flex-auto flex flex-col"
          >
            <component :is="Component" />
          </div>
        </keep-alive>
      </transition>
    </router-view>
  </div>
</template>
<script setup>
import { useRoute, onBeforeRouteUpdate } from 'vue-router';
import { ref } from 'vue';
import { useStore } from 'vuex';
let store = useStore();
store.dispatch('getPayTime');
let box = ref(false);

onBeforeRouteUpdate((to, from) => {
  if (to.meta.transition && from.meta.transition) {
    box.value = true;
  } else {
    box.value = false;
  }
});
</script>
<style lang="less" scoped>
.skewDown-enter-active,
.skewDown-leave-active,
.skewUp-enter-active,
.skewUp-leave-active {
  position: absolute !important;
  width: 100%;
  left: 0;
  transition: transform 0.5s ease, opacity 0.05s linear;
  transform-style: preserve-3d;
}

.skewUp-leave-to {
  animation: upLeave 0.5s ease-in-out;
  transform-origin: left;
}
@keyframes upLeave {
  0% {
    z-index: 3;
    transform: rotateY(0deg) scale(1);
  }
  50% {
    z-index: 3;
    transform: rotateY(60deg) scale(0.8);
  }
  51% {
    z-index: 2;
    transform: rotateY(60deg) scale(0.8);
  }
  100% {
    z-index: 2;
    transform: rotateY(0deg) scale(1);
  }
}
.skewUp-enter-from {
  opacity: 0;
}
.skewUp-enter-to {
  animation: upEnter 0.5s ease-in-out;
  transform-origin: right;
  opacity: 1;
}
@keyframes upEnter {
  0% {
    z-index: 2;
    transform: rotateY(0deg) scale(1);
  }
  50% {
    z-index: 2;
    transform: rotateY(-60deg) scale(0.8);
  }
  51% {
    z-index: 3;
    transform: rotateY(-60deg) scale(0.8);
  }
  100% {
    z-index: 3;
    transform: rotateY(0deg) scale(1);
  }
}
.skewDown-enter-from {
  opacity: 0;
}
.skewDown-enter-to {
  animation: downEnter 0.5s ease-in-out;
  transform-origin: left;
  opacity: 1;
}
@keyframes downEnter {
  0% {
    z-index: 2;
    transform: rotateY(0deg) scale(1);
  }
  50% {
    z-index: 2;
    transform: rotateY(60deg) scale(0.8);
  }
  51% {
    z-index: 3;
    transform: rotateY(60deg) scale(0.8);
  }
  100% {
    z-index: 3;
    transform: rotateY(0deg) scale(1);
  }
}
.skewDown-leave-to {
  transform-origin: right;
  animation: downLeave 0.5s ease-in-out;
}
@keyframes downLeave {
  0% {
    z-index: 3;
    transform: rotateY(0deg) scale(1);
  }
  50% {
    z-index: 3;
    transform: rotateY(-60deg) scale(0.8);
  }
  51% {
    z-index: 2;
    transform: rotateY(-60deg) scale(0.8);
  }
  100% {
    z-index: 2;
    transform: rotateY(0deg) scale(1);
  }
}
.userLayout {
  position: relative;
}
.perspective {
  perspective: 1000px;
}
</style>
