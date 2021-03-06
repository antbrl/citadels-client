<template>
<div class="py-2 d-flex justify-content-start align-items-end overflow-auto">
  <div v-if="board.crown" class="crown card rounded-pill bg-danger p-3 m-2 shadow-sm">👑</div>
  <div class="mr-auto"></div>
  <DistrictCard
    v-for="id, i in board.hand"
    :key="i"
    :district-id="id"
    class="mr-2"
    :disabled="showTmpHand || (buildMode && !canBuild(id))"
    :selectable="canBuild(id) || discardCardsMode || laboratoryMode"
    v-model:selected="selectedCards[i]"
  />
  <div
    v-if="showTmpHand"
    class="bg-light d-flex justify-content-start pl-2 py-2 my-n2"
  >
    <DistrictCard
      v-for="id, i in board.tmpHand"
      :key="i"
      :district-id="id"
      class="mr-2"
      @click="chooseCard(id)"
      :selectable="showTmpHand"
    />
  </div>
  <div
    class="stash d-flex flex-column-reverse flex-wrap-reverse justify-content-start ml-auto"
  ><span v-for="i in board.stash" :key="i" class="coin">🪙</span></div>
</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { store } from '../../../store';
import { Move, MoveType } from '../../../types/gameTypes';
import DistrictCard from './DistrictCard.vue';

export default defineComponent({
  name: 'PlayerHand',
  components: {
    DistrictCard,
  },
  props: {
    board: {
      required: true,
    },
    buildMode: {
      type: Boolean,
      default: false,
    },
    discardCardsMode: {
      type: Boolean,
      default: false,
    },
    laboratoryMode: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      selectedCards: [] as boolean[],
    };
  },
  computed: {
    showTmpHand() {
      return this.board.tmpHand.length > 0;
    },
  },
  methods: {
    canBuild(name: string): boolean {
      if (!this.buildMode) return false;
      return !this.board.city.includes(name)
        && store.getters.getDistrictFromId(name).cost <= this.board.stash;
    },
    async chooseCard(name: string) {
      try {
        const move: Move = { type: MoveType.DRAW_CARDS, data: name };
        await store.dispatch('sendMove', move);
      } catch (error) {
        console.log('error when sending move', error);
      }
    },
    async chooseCardBuild(name: string) {
      if (!this.canBuild(name)) return;

      try {
        const move: Move = { type: MoveType.BUILD_DISTRICT, data: name };
        await store.dispatch('sendMove', move);
      } catch (error) {
        console.log('error when sending move', error);
      }
    },
    async chooseCardLaboratory(name: string) {
      try {
        const move: Move = { type: MoveType.LABORATORY_DISCARD_CARD, data: name };
        await store.dispatch('sendMove', move);
      } catch (error) {
        console.log('error when sending move', error);
      }
    },
  },
  watch: {
    board: {
      handler(newBoard, oldBoard) {
        if (newBoard.hand !== oldBoard.hand) {
          this.selectedCards = [];
        }
      },
      deep: true,
    },
    selectedCards: {
      handler(val) {
        const cards: string[] = [];
        val.forEach((isSelected, index) => {
          if (isSelected) {
            const card = this.board.hand[index];
            if (card !== undefined) cards.push(card);
          }
        });

        store.commit('setSelectedCards', { cards });

        if (cards.length > 0) {
          if (this.buildMode) {
            this.chooseCardBuild(cards[0]);
            this.selectedCards = [];
          } else if (this.laboratoryMode) {
            this.chooseCardLaboratory(cards[0]);
            this.selectedCards = [];
          }
        }
      },
      deep: true,
    },
  },
});
</script>

<style lang="scss" scoped>
.stash {
  height: 9em;
}

.coin {
  font-size: 2em;
  height: .8em;
  position: relative;
  bottom: .8em;
  text-shadow: 0 -1px 2px rgba(0, 0, 0, .2);
}

.crown {
  font-size: 2em;
  min-width: 2.5em;
  text-align: center;
}
</style>
