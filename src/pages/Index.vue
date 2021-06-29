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
      <p class="text-h6 q-mb-none">You have completed</p>

      <span
        :class="{ 'rep-count--complete': visableRepsComplete }"
        class="rep-count"
      >
        {{ chain[visableBlockIndex]?.reps }} / {{ repsPerBlock }}
      </span>
      <div class="emotion-and-date">
        <span class="emotion-and-date__emoji">
          {{ chain[visableBlockIndex]?.emotion }}
        </span>
        <span class="emotion-and-date__date">
          {{ visableBlockDate }}
        </span>
      </div>

      <div v-if="emotionInputActive" class="text-center">
        <p>How do you feel?</p>
        <div class="row q-gutter-md">
          <q-btn
            v-for="emotion in emotions"
            :key="emotion"
            unelevated
            color="grey-10"
            @click="addEmotion(emotion)"
          >
            {{ emotion }}
          </q-btn>
        </div>
      </div>

      <div v-else class="text-center">
        <p>Add some reps</p>
        <div class="row q-gutter-md">
          <q-btn
            v-for="reps in repsBtns"
            :key="reps"
            unelevated
            color="grey-10"
            @click="addReps(reps)"
          >
            {{ reps }}
            <q-menu touch-position context-menu>
              <q-list dense style="min-width: 100px">
                <q-item clickable v-close-popup @click="delReps(reps)">
                  <q-item-section>Subtract</q-item-section>
                </q-item>
              </q-list>
            </q-menu>
          </q-btn>
        </div>
      </div>

      <div style="width: 50%" class="text-center">
        <p>Overall progress</p>
        <q-linear-progress
          stripe
          rounded
          size="18px"
          :value="chainProgress / totalChainReps"
          color="positive"
          class="q-mb-sm"
        />
        <div>{{ chainProgress }} / {{ totalChainReps }}</div>
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
  emotion?: string;
  comment?: string;
}

const repsBtns = [10, 20, 30, 40];
const emotions = ['💩', '🤡', '😎', '🦄'];

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

    const emotionInputActive = ref(false);
    watchEffect(() => {
      if (
        chain.value[visableBlockIndex.value].reps >= repsPerBlock.value &&
        !chain.value[visableBlockIndex.value].emotion
      ) {
        console.log('emotion');
        emotionInputActive.value = true;
      }
    });

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

    function delReps(amount: number) {
      chain.value[visableBlockIndex.value].reps -= amount;
    }

    function addEmotion(emotion: string) {
      chain.value[visableBlockIndex.value].emotion = emotion;
      emotionInputActive.value = false;
    }

    return {
      repsBtns,
      emotions,
      chain,
      chainLength,
      repsPerBlock,
      todaysBlockIndex,
      visableBlockDate,
      visableRepsComplete,
      visableBlockIndex,
      chainProgress,
      totalChainReps,
      emotionInputActive,
      goToBlockIndex,
      createNewChain,
      addReps,
      delReps,
      addEmotion,
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

.emotion-and-date {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.emotion-and-date__emoji {
  font-size: 1.5rem;
}
.emotion-and-date__date {
  font-weight: bold;
}
</style>
