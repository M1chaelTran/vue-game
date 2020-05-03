<template>
  <div class="row items-center">
    <span :class="getCharacterColor(event.pitcher)"
      >{{ event.pitcher.name }}
    </span>
    &nbsp;<small>({{ event.pitcher.health }})</small> &nbsp;{{
      actionText
    }}&nbsp;
    <span :class="getCharacterColor(event.receiver)">{{
      event.receiver.name
    }}</span>
    &nbsp;
    <span :class="actionColor">{{ actionOutcomeText }}</span>
  </div>
</template>

<script>
import { TEAM_TYPES, ATTACK_TYPES } from "pages/constant";
export default {
  name: "Event",
  props: ["event"],
  computed: {
    actionColor() {
      if (this.event.actionType === ATTACK_TYPES.special)
        return "text-red text-bold";
      else if (this.event.actionType === ATTACK_TYPES.heal)
        return "text-green text-bold";
      else return "";
    },
    actionText() {
      return this.event.actionType === ATTACK_TYPES.heal
        ? "healed"
        : "attacked";
    },
    actionIcon() {
      let icon = "⚔️";
      switch (this.event.actionType) {
        case ATTACK_TYPES.heal:
          icon = "💚";
          break;
        case ATTACK_TYPES.special:
          icon = "🌟";
          break;
        default:
          console.error("Unhandled action type", this.event.actionType);
          break;
      }
      return icon;
    },
    actionOutcomeText() {
      let outcomeText = `dealing ${this.event.strength} ${this.actionIcon} damages`;
      if (this.event.strength === 0) outcomeText = "but missed";
      else if (this.event.actionType === ATTACK_TYPES.heal)
        outcomeText = `+${this.event.strength} ${this.actionIcon} health`;
      return outcomeText;
    }
  },
  methods: {
    getCharacterColor(character) {
      return character.team === TEAM_TYPES.player ? "text-blue" : "text-orange";
    }
  }
};
</script>
