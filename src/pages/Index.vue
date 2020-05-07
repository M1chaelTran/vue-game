<template>
  <div class="p-4 md:p-8">
    <div class="flex mb-4 md:mb-8 space-x-4 md:space-x-8">
      <div class="flex flex-1 flex-col items-center">
        <character
          :key="player.id"
          v-for="(player, index) in players"
          :character="player"
          :index="index"
          :gameStarted="gameStarted"
          @removePlayer="removePlayer"
        />
        <q-btn v-if="!maxPlayer && !gameStarted" outline @click="addPlayer"
          >Add allies</q-btn
        >
      </div>
      <div class="flex flex-1 flex-col items-center">
        <character
          :key="monster.id"
          v-for="monster in monsters"
          :character="monster"
        />
      </div>
    </div>
    <div class="q-mt-lg ">
      <controls
        :started="gameStarted"
        @startGame="startGame"
        @attack="attack"
        @specialAttack="attack(true)"
        @heal="heal"
        @giveUp="giveUp"
        :difficulty="difficulty"
        @setDifficulty="difficulty = $event"
      />
    </div>

    <div class="q-mt-xl flex column items-center">
      <q-list
        bordered
        separator
        class="q-mb-md"
        :key="i"
        v-for="(round, i) in rounds.slice().reverse()"
      >
        <q-item-label header class="bg-gray-200"
          >Round {{ rounds.length - i }}</q-item-label
        >
        <q-item :key="index" v-for="(event, index) in round">
          <event :event="event" />
        </q-item>
      </q-list>
    </div>

    <outcome
      :showGameOutcome="showGameOutcome"
      :gameOutcome="gameOutcome"
      @startGame="startGame"
      @closeGameOutcome="closeGameOutcome"
    />
  </div>
</template>

<script>
import { chain, filter, forEach, isEmpty, reduce } from "lodash";
import { uid } from "quasar";
import faker from "faker";
import {
  ATTACK_TYPES,
  GAME_DIFFICULTY,
  SPECIAL_ATTACK_MULTIPLIER,
  TEAM_TYPES
} from "pages/constant";

const getDefaultCharacter = () => ({
  health: 100,
  attack: {
    min: 0,
    max: 10
  }
});

export default {
  name: "PageIndex",
  data() {
    return {
      characters: {
        id1: {
          id: "id1",
          name: "you",
          health: 100,
          attack: {
            min: 0,
            max: 10
          },
          team: TEAM_TYPES.player
        },
        id3: {
          id: "id3",
          name: "monster",
          health: 100,
          attack: {
            min: 0,
            max: 10
          },
          team: TEAM_TYPES.enemy
        }
      },
      gameStarted: false,
      rounds: [],
      alivePlayers: [],
      aliveMonster: [],
      showGameOutcome: false,
      gameOutcome: null,
      difficulty: "normal"
    };
  },
  computed: {
    players() {
      return filter(this.characters, x => x.team === TEAM_TYPES.player);
    },
    monsters() {
      return filter(this.characters, x => x.team === TEAM_TYPES.enemy);
    },
    maxPlayer() {
      return this.players.length === 3;
    }
  },
  watch: {
    gameOutcome: function(val) {
      if (!!val) this.showGameOutcome = true;
      // const vm = this;
      // // reset back after 5seconds
      // setTimeout(() => {
      //   vm.showGameOutcome = false;
      // }, 5000);
    },
    difficulty: function(val) {
      this.setDifficulty(val);
    }
  },
  methods: {
    startGame() {
      this.gameStarted = true;
      // reset all characters health
      this.resetCharacterHealth();
      this.rounds = [];
      this.gameOutcome = null;
      this.showGameOutcome = false;
    },
    attack(specialAttack = false) {
      // create a stack of randomize alived characters
      const characterStack = chain(this.characters)
        .filter(x => x.health > 0)
        .shuffle()
        .value();
      const roundEvents = this.performAttack(characterStack, specialAttack);

      // log round events
      this.rounds.push(roundEvents);

      this.checkGameState();
    },
    performAttack(characterStack, specialAttack) {
      const roundEvents = [];
      // for each alive character
      // attack a random opposing team character
      forEach(characterStack, x => {
        // character might have die during the fight, so skip over it
        if (x.health === 0) return;

        const randomAttackStrength = getRandomStrength(x.attack, specialAttack);
        const randomEnemy = chain(this.characters)
          .filter(y => y.team !== x.team && y.health > 0)
          .shuffle()
          .head()
          .value();
        if (isEmpty(randomEnemy, true)) return;

        randomEnemy.health =
          randomEnemy.health > randomAttackStrength
            ? randomEnemy.health - randomAttackStrength
            : 0;

        roundEvents.push({
          pitcher: { ...x },
          actionType: specialAttack
            ? ATTACK_TYPES.special
            : ATTACK_TYPES.attack,
          strength: randomAttackStrength,
          receiver: { ...randomEnemy }
        });
      });

      return roundEvents;
    },
    checkGameState() {
      const alivePlayers = this.players.filter(x => x.health > 0);
      const aliveMonster = this.monsters.filter(x => x.health > 0);
      // game end when there's no more player or monster alive
      if (!alivePlayers.length || !aliveMonster.length) {
        this.gameStarted = false;
        this.gameOutcome = alivePlayers.length ? "win" : "lost";
      }
    },
    heal() {
      // player at disadvantage here since the enemies will attack first
      const characterStack = chain(this.characters)
        .filter(x => x.team === TEAM_TYPES.enemy && x.health > 0)
        .shuffle()
        .value();
      const attackEvents = this.performAttack(characterStack, false);

      // player heals
      // 1. for each player alive
      // 2. check to see which player has lowest health
      // 3. heal that player
      let alivePlayersWithLowestHealth = chain(this.characters)
        .filter(x => x.team === TEAM_TYPES.player && x.health > 0)
        .orderBy(["health"], ["asc"])
        .value();

      const roundEvents = [];
      alivePlayersWithLowestHealth.forEach(x => {
        const randomHealStrength = getRandomStrength(x.attack);
        let healPerformed = false;
        const pitcher = { ...x };
        alivePlayersWithLowestHealth.forEach(y => {
          if (!healPerformed && y.health <= x.health) {
            const totalHeal = y.health + randomHealStrength;
            y.health = totalHeal > 100 ? 100 : totalHeal;
            healPerformed = true;

            roundEvents.push({
              pitcher,
              actionType: ATTACK_TYPES.heal,
              strength: randomHealStrength,
              receiver: { ...y }
            });
          }
        });
      });

      // log round events
      this.rounds.push([...attackEvents, ...roundEvents]);

      this.checkGameState();
    },
    addPlayer() {
      const id = uid();
      this.characters = {
        ...this.characters,
        [id]: {
          ...getDefaultCharacter(),
          id,
          name: faker.name.findName(),
          team: TEAM_TYPES.player
        }
      };
      this.setDifficulty(this.difficulty);
    },
    removePlayer(playerId) {
      this.characters = filter(this.characters, x => x.id !== playerId);
      this.setDifficulty(this.difficulty);
    },
    setDifficulty(difficulty) {
      // easy or normal difficulty
      let numberOfMonsters = this.players.length;

      if (difficulty === GAME_DIFFICULTY.hard)
        numberOfMonsters = this.players.length + 1;
      else if (difficulty === GAME_DIFFICULTY.hardcore)
        numberOfMonsters = this.players.length + 2;

      const enemies = reduce(
        Array(numberOfMonsters).fill(),
        acc => {
          const id = uid();
          acc[id] = {
            ...getDefaultCharacter(),
            id,
            name: faker.random.word(),
            team: TEAM_TYPES.enemy
          };
          return acc;
        },
        {}
      );

      if (difficulty === GAME_DIFFICULTY.easy) {
        // half the enemy max attack strength
        forEach(enemies, x => {
          x.health = Math.floor(x.health / 1.2);
        });
      }
      this.characters = {
        ...this.players,
        ...enemies
      };
    },
    giveUp() {
      this.gameStarted = false;
      this.gameOutcome = "lost";
    },
    closeGameOutcome() {
      this.showGameOutcome = false;
    },
    resetCharacterHealth() {
      forEach(this.characters, x => {
        if (
          this.difficulty === GAME_DIFFICULTY.easy &&
          x.team === TEAM_TYPES.enemy
        )
          x.health = Math.floor(100 / 1.2);
        else x.health = 100;
      });
    }
  },
  components: {
    character: require("components/Character").default,
    controls: require("components/Controls").default,
    event: require("components/Event").default,
    outcome: require("components/Outcome").default
  }
};

function getRandomInt(min, max) {
  min = Math.ceil(min);
  max = Math.floor(max);
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function getRandomStrength(attack, isSpecialAttack) {
  let min = attack.min;
  let max = attack.max;
  if (isSpecialAttack) {
    min = max;
    max = max * SPECIAL_ATTACK_MULTIPLIER;
  }
  return getRandomInt(min, max);
}
</script>
