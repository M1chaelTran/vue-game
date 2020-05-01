<template>
  <q-linear-progress size="50px" :value="health" :color="healthColor">
    <div class="absolute-full flex flex-center">
      <q-badge color="white" text-color="accent" :label="healthPercentage" />
    </div>
  </q-linear-progress>
</template>

<script>
export default {
  name: "HealBar",
  props: ["health"],
  data() {
    return {};
  },
  computed: {
    healthPercentage() {
      return (this.health * 100).toFixed(0) + "%";
    },
    healthColor() {
      return this.health < 0.1 ? "red" : "green";
    }
  }
};
</script>
