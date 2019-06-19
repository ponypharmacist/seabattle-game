<template lang="pug">
  
  .field-grid
    template
      template(v-for="(row, indexRow) in this.getFieldByParams(this.player)")
        div(v-for="(cell, indexCell) in row"
            @click="fireCannon(indexRow, indexCell)"
            :class="{ isShip: cell.ship, mine: cell.mine, dead: cell.dead }")

</template>

<script>
import { mapGetters, mapMutations } from 'vuex'

export default {
  name: 'FieldGridOpponent',
  props: {
    player: String,
  },

  computed: {
    ...mapGetters([
      'isShotsLeft',
      'getFieldByParams',
      'opponentFieldCheck',
      'getDamagedShip',
      'isShipDead',
      'checkWinCondition',
    ]),
  },

  methods: {
    ...mapMutations([
      'sendAlertMessage',
      'placeMine',
      'markShipDamaged',
      'markShipDead',
      'placeDeadTiles',
      'reduceShotsAvailable',
      'setLastShot',
      'addToLastShots',
    ]),

    fireCannon (row, col) {
      let alertResult = 'You\'ve missed.'
      // 0. Check if any shots left for this turn
      if (!this.isShotsLeft) {
        alertResult = 'Pirate budget allows only one shot per turn, captain.'
        this.sendAlertMessage(alertResult)
        return
      }
      // 0. Place a mine mark
      if ( this.opponentFieldCheck(row, col, 'mine') || this.opponentFieldCheck(row, col, 'dead') ) {
        alertResult = 'We\'ve already shot here, captain.'
      } else {
        this.placeMine({row: row, col: col})
        this.reduceShotsAvailable()
        this.setLastShot({row: row + 1, col: col + 1})
        this.addToLastShots({row: row, col: col})
      }
      // 1. Is this a hit?
      if ( this.opponentFieldCheck(row, col, 'ship') ) {
        // 2. Mark ship as isDamaged
        this.markShipDamaged({row: row, col: col})
        alertResult = 'Enemy ship hit!'
        // 3. Is this ship completely dead?
        let damagedShip = this.getDamagedShip(row, col)
        if ( this.isShipDead(damagedShip) ) {
          this.markShipDead(damagedShip)
          this.placeDeadTiles(damagedShip)
          alertResult = 'Enemy ship down! Rape the treasure and pillage the ladies!'
          if ( this.checkWinCondition ) {
            alertResult = 'All yer foes r ded. Ye be da best captn in these waters!'
          }
        }
      }
      // 4. Display alert
      this.sendAlertMessage(alertResult)
    },

  }

}
</script>

<style lang="sass">
</style>
