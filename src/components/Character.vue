<template>
  <div class="w-full">
    <h4 class="text-center q-mb-sm text-3xl">
      <span :class="{ 'text-strike': isDead }">{{ character.name }}</span>
      {{ isDead ? "☠️" : character.team === "enemy" ? "🤖" : "" }}
    </h4>
    <health-bar :health="character.health / 100" />
  </div>
</template>

<script>
export default {
  name: "Character",
  props: ["character"],
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
