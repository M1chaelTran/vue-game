<template>
  <q-dialog v-model="showGameOutcome" persistent>
    <q-card>
      <q-card-section class="q-pt-none">
        <img :src="gameOutcome && require(`assets/${gameOutcome}.png`)" />
      </q-card-section>

      <q-card-actions align="right">
        <q-btn label="Close" outline v-close-popup />
        <q-btn
          label="Another round?"
          color="primary"
          v-close-popup
          @click="$emit('startGame')"
        />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script>
export default {
  name: "GameOutcome",
  props: ["gameOutcome", "showGameOutcome"]
};
</script>
