<template>
  <div class="flex flex-col items-center border rounded p-4" v-if="started">
    <div
      class="flex flex-1 flex-no-wrap justify-center items-center space-x-2 md:space-x-4"
    >
      <q-btn color="primary" @click="$emit('attack')">Attack</q-btn>
      <q-btn color="red-5" @click="$emit('specialAttack')"
        >Special attack</q-btn
      >
      <q-btn color="green-5" @click="$emit('heal')">Heal</q-btn>
    </div>
    <q-btn class="mt-8" outline @click="$emit('giveUp')">Give up</q-btn>
  </div>
  <div v-else class="flex flex-col items-center border rounded p-4">
    <h4 class="text-xl m-0">Difficulty</h4>
    <div class="flex space-x-4 mb-8 flex-no-wrap">
      <q-btn
        v-for="mode in gameModes"
        :key="mode"
        :outline="!selected(mode)"
        :color="selected(mode) ? 'accent' : ''"
        @click="$emit('setDifficulty', mode)"
        >{{ mode }}</q-btn
      >
    </div>
    <q-btn class="h-16" color="primary" @click="$emit('startGame')"
      >Start game</q-btn
    >
  </div>
</template>

<script>
import { GAME_DIFFICULTY } from "pages/constant";

export default {
  name: "Controls",
  props: ["started", "difficulty"],
  data() {
    return {
      gameModes: GAME_DIFFICULTY
    };
  },
  methods: {
    selected(mode) {
      return this.difficulty === mode;
    }
  }
};
</script>
