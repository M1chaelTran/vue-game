<template>
  <div class="mb-16">
    <h4 class="text-center q-mb-sm text-3xl">
      <span :class="{ 'text-strike': isDead }">{{ character.name }}</span>
      {{ isDead ? "☠️" : "" }}
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
