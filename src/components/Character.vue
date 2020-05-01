<template>
  <div class="w-full mb-8">
    <h4 class="text-center text-3xl">
      <span :class="{ 'text-strike': isDead }">{{ character.name }}</span>
      {{ isDead ? "☠️" : character.team === "enemy" ? "🤖" : "" }}
      <q-btn
        flat
        round
        color="red-4"
        v-if="index > 0 && !gameStarted"
        icon="delete"
        @click="$emit('removePlayer', character.id)"
      />
    </h4>
    <health-bar :health="character.health / 100" />
  </div>
</template>

<script>
export default {
  name: "Character",
  props: ["character", "index", "removePlayer", "gameStarted"],
  computed: {
    isDead() {
      return this.character.health === 0;
    }
  },
  components: {
    "health-bar": require("components/HealthBar").default
  }
};
</script>
