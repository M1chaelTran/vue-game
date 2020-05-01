<template>
  <div class="row justify-evenly" v-if="started">
    <q-btn color="primary" @click="$emit('attack')">Attack</q-btn>
    <q-btn color="red-5" @click="$emit('specialAttack')">Special attack</q-btn>
    <q-btn color="green-5" @click="$emit('heal')">Heal</q-btn>
    <q-btn outline @click="$emit('giveUp')">Give up</q-btn>
  </div>
  <div v-else class="row justify-center">
    <q-btn color="primary" @click="$emit('startGame')">Start new game</q-btn>
  </div>
</template>

<script>
export default {
  name: "Controls",
  props: ["started"]
};
</script>
