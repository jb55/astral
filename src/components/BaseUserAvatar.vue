<template>
  <div :class='(bordered ? "bordered-avatar" : "") + (hoverEffect ? " hovered-avatar" : "")'>
    <q-avatar :rounded='!round' class='relative-position' :size='size' @click.stop="toProfile(pubkey)">
      <img :src="$store.getters.avatar(pubkey)" />
      <div :class='alignRight ? "icon-right" : "icon-left"'>
      <BaseButtonNIP05
        v-if='showVerified'
        :pubkey='pubkey'
      />
      </div>
    </q-avatar>
  </div>
</template>

<script>
import helpersMixin from '../utils/mixin'
import BaseButtonNIP05 from 'components/BaseButtonNIP05.vue'

export default {
  mixins: [helpersMixin],
  components: {
    BaseButtonNIP05,
  },
  props: {
    pubkey: {type: String, required: true},
    alignRight: {type: Boolean, default: false},
    size: {type: String, default: ''},
    round: {type: Boolean, default: false},
    bordered: {type: Boolean, default: false},
    showVerified: {type: Boolean, default: false},
    hoverEffect: {type: Boolean, default: false},
  },
}
</script>

<style lang='scss' scoped>
.bordered-avatar .q-avatar img {
  border: 2px solid $accent;
  background: $dark;
}
.hovered-avatar .q-avatar:hover {
  transform: scale(1.5) translateX(.25rem);
}
.icon-right {
  position: absolute;
  top: -.4rem;
  right: -.5rem;
}
.icon-left {
  position: absolute;
  top: -.4rem;
  left: -.5rem;
}
</style>
