<template>
  <q-page padding class="page-grid">
    <q-btn
      v-if="!chain.length"
      unelevated
      class="glossy"
      color="primary"
      size="xl"
      padding="8px 32px"
      no-caps
      @click="createNewChain"
    >
      Start
    </q-btn>

    <template v-else>
      <p>You have completed</p>

      <span
        :class="{ 'rep-count--complete': visableRepsComplete }"
        class="rep-count"
      >
        {{ chain[visableBlockIndex]?.reps }} / {{ repsPerBlock }}
      </span>

      {{ visableBlockDate }}
      <div class="text-center">
        <p>Add some reps</p>
        <div class="row q-gutter-md">
          <q-btn unelevated color="grey-10" @click="addReps(10)">10</q-btn>
          <q-btn unelevated color="grey-10" @click="addReps(20)">20</q-btn>
          <q-btn unelevated color="grey-10" @click="addReps(30)">30</q-btn>
          <q-btn unelevated color="grey-10" @click="addReps(40)">40</q-btn>
        </div>
      </div>

      <div style="width: 50%" class="text-center">
        <!-- <div>{{ chainProgress }} / {{ totalChainReps }}</div> -->
        <q-linear-progress
          stripe
          rounded
          size="18px"
          :value="chainProgress / totalChainReps"
          color="positive"
          class="q-mb-sm"
        />
        <p>Overall progress</p>
      </div>

      <div style="margin-top: auto" class="full-width row justify-between">
        <q-btn
          :class="{ invisable: !visableBlockIndex }"
          flat
          @click="goToBlockIndex(visableBlockIndex - 1)"
        >
          <div class="text-center">
            <q-icon name="chevron_left" />
            <br />
            {{ visableBlockIndex }}
          </div>
        </q-btn>
        <q-btn
          :class="{ invisable: visableBlockDate === 'Today' }"
          flat
          no-caps
          @click="goToBlockIndex(todaysBlockIndex)"
        >
          <div class="text-center">
            <q-icon name="radio_button_unchecked" />
            <br />
            Today
          </div>
        </q-btn>
        <q-btn flat @click="goToBlockIndex(visableBlockIndex + 1)">
          <div class="text-center">
            <q-icon name="chevron_right" />
            <br />
            {{ chain.length - 1 - visableBlockIndex }}
          </div>
        </q-btn>
      </div>
    </template>
  </q-page>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watchEffect } from 'vue';
import startOfToday from 'date-fns/startOfToday';
import isToday from 'date-fns/isToday';
import format from 'date-fns/format';
import { LocalStorage } from 'quasar';

interface Chain {
  day: number;
  reps: number;
}

export default defineComponent({
  name: 'PageIndex',

  setup() {
    const chainLength = ref(100);
    const repsPerBlock = ref(100);
    const chain = ref<Chain[]>([]);

    if (LocalStorage.has('pushups')) {
      chain.value = LocalStorage.getItem('pushups') as Chain[];
    }

    watchEffect(() => LocalStorage.set('pushups', chain.value));

    function createNewChain() {
      const newChain = [];
      for (let i = 0; i < chainLength.value; i++) {
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
      if (index < 0 || index === chainLength.value) return;
      visableBlockIndex.value = index;
    }

    const visableBlockDate = computed(() => {
      if (visableBlockIndex.value === -1) return '';
      if (isToday(chain.value[visableBlockIndex.value].day)) {
        return 'Today';
      } else {
        return format(
          chain.value[visableBlockIndex.value].day,
          'EEEE d MMMM yyyy'
        );
      }
    });

    const visableRepsComplete = computed(() => {
      return chain.value[visableBlockIndex.value]?.reps >= repsPerBlock.value;
    });

    const chainProgress = computed(() => {
      return chain.value.reduce((acc, curr) => {
        return acc + curr.reps;
      }, 0);
    });

    const totalChainReps = computed(
      () => chainLength.value * repsPerBlock.value
    );

    function addReps(amount: number) {
      chain.value[visableBlockIndex.value].reps += amount;
    }

    return {
      chain,
      chainLength,
      repsPerBlock,
      todaysBlockIndex,
      visableBlockDate,
      visableRepsComplete,
      visableBlockIndex,
      chainProgress,
      totalChainReps,
      goToBlockIndex,
      createNewChain,
      addReps,
      isToday,
    };
  },
});
</script>

<style lang="scss" scoped>
.page-grid {
  display: grid;
  place-items: center;
}
.rep-count {
  font-weight: bold;
  font-size: 2rem;
}

.rep-count--complete {
  color: $green;
}

.invisable {
  opacity: 0;
}
</style>
