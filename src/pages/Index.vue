<template>
  <q-page class="row items-center justify-evenly">
    <q-btn v-if="!chain.length" @click="createNewChain">START</q-btn>
    <!-- {{ chain }} -->
    {{ visableBlockDate }}

    <span
      :class="{ 'rep-count--complete': visableRepsComplete }"
      class="rep-count"
      >{{ chain[visableBlockIndex]?.reps }}</span
    >
    <q-btn @click="addReps(10)">+10</q-btn>
    <q-btn @click="addReps(20)">+20</q-btn>
    <q-btn @click="addReps(30)">+30</q-btn>
    <q-btn @click="addReps(40)">+40</q-btn>
    <br />
    <q-btn @click="goToBlockIndex(visableBlockIndex - 1)"
      >prev{{ visableBlockIndex }}</q-btn
    >
    <q-btn @click="goToBlockIndex(todaysBlockIndex)">TODAY</q-btn>
    <q-btn @click="goToBlockIndex(visableBlockIndex + 1)"
      >next{{ chain.length - 1 - visableBlockIndex }}</q-btn
    >
  </q-page>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watchEffect } from 'vue';
import startOfToday from 'date-fns/startOfToday';
import format from 'date-fns/format';
import { LocalStorage } from 'quasar';

interface Chain {
  day: number;
  reps: number;
}

export default defineComponent({
  name: 'PageIndex',

  setup() {
    const chain = ref<Chain[]>([]);

    if (LocalStorage.has('pushups')) {
      chain.value = LocalStorage.getItem('pushups') as Chain[];
    }

    watchEffect(() => LocalStorage.set('pushups', chain.value));

    function createNewChain() {
      const newChain = [];
      for (let i = 0; i < 100; i++) {
        const day: number = newChain.length
          ? newChain[newChain.length - 1].day + 86400000
          : startOfToday().getTime();
        newChain.push({ day, reps: 0 });
      }
      chain.value = newChain;
      LocalStorage.set('pushups', newChain);
      visableBlockIndex.value = 0;
    }

    const todaysBlockIndex = computed(() => {
      const today = startOfToday().getTime();
      return chain.value.findIndex((block) => block.day === today);
    });

    const visableBlockIndex = ref(todaysBlockIndex.value);

    function goToBlockIndex(index: number) {
      if (index < 0 || index === 100) return;
      visableBlockIndex.value = index;
    }

    const visableBlockDate = computed(() => {
      if (visableBlockIndex.value === -1) return '';
      return format(
        chain.value[visableBlockIndex.value]?.day,
        'EEEE d MMMM yyyy'
      );
    });

    const visableRepsComplete = computed(() => {
      return chain.value[visableBlockIndex.value]?.reps >= 100;
    });

    function addReps(amount: number) {
      chain.value[visableBlockIndex.value].reps += amount;
    }

    return {
      chain,
      todaysBlockIndex,
      visableBlockDate,
      visableRepsComplete,
      visableBlockIndex,
      goToBlockIndex,
      createNewChain,
      addReps,
    };
  },
});
</script>

<style lang="scss" scoped>
.rep-count {
  font-weight: bold;
  font-size: 2rem;
}

.rep-count--complete {
  color: $green;
}
</style>
