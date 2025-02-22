<template>
  <q-list ref='thread'>
    <div
      v-for="(event, index) in filledEvents"
      :key="event.id + index"
      class='no-padding'
    >
      <BaseShowMore v-if="event.id === 'FILLER'" :root="event.root" :replies='event.replies ? event.replies : []'/>
      <BasePost
        v-else
        :event="event"
        :position="position(index)"
        :reply-depth='replyDepth'
        @resized='resize'
        @add-event='addEvent'
        @click.stop="toEvent(event.id)"
      />
    </div>
  </q-list>
</template>

<script>
import helpersMixin from '../utils/mixin'
import BaseShowMore from 'components/BaseShowMore'
// import {getEventTagWithRelay} from '../utils/helpers'

export default {
  name: 'BasePostThread',
  emits: ['resized', 'add-event'],
  mixins: [helpersMixin],
  props: {
    events: {type: Array, required: true},
    isAncestors: {type: Boolean, default: false},
    replyDepth: {type: Number, default: 0},
  },
  components: {
    BaseShowMore,
  },

  data() {
    return {
      resizing: false,
    }
  },

  computed: {
    root() {
      return this.events[0].id
    },
    threadWidth() {
      // only using resizing to trigger compute
      if (this.resizing) return this.$refs.thread?.$el?.clientWidth
      return this.$refs.thread?.$el?.clientWidth
    },

    filledEvents() {
      if (this.resizing && !this.resizing) return
      if (this.events.length === 0) return []

      var filled = [this.events[0]]
      if (this.events.length === 1) return filled

      for (let i = 1; i < this.events.length; i++) {
        let curr = this.events[i]
        let prev = this.events[i - 1]
        // if (curr.replies && curr.replies.length) console.log('curr: ', curr)
        let currEventTags = curr.tags.filter(([t, v]) => t === 'e' && v).map(([_, v]) => v)
        if (currEventTags[currEventTags.length - 1] !== prev.id) {
        // if (curr.tags[curr.tags.length - 1][1] !== prev.id) {
          filled.push({id: 'FILLER', root: prev.id})
        }

        if ((i === this.events.length - 1) && curr.replies?.length && this.threadWidth &&
          // (this.replyDepth >= 5)) {
          (this.replyDepth >= 5 || (this.replyDepth > 0 && this.threadWidth < 175))) {
            let replies = Array.from(curr.replies)
            let event = Object.assign({}, curr)
            event.replies = []
            // curr.replies = []
            filled.push(event)
            filled.push({id: 'FILLER', root: curr.id, replies: replies})
            // filled.concat([curr, {id: 'FILLER', root: curr.id, replies: replies}])
            // console.log('filled', filled)
        } else filled.push(curr)
      }

      return filled
    },
  },


  methods: {
    position(index) {
      if (!this.isAncestors) {
        // normal thread
        if (this.filledEvents.length === 1) return 'single'
        if (index === 0) return 'first'
        if (index === this.filledEvents.length - 1) return 'last'
        return 'middle'
      } else {
        // in this mode the last event should have the left bar to the bottom,
        // as it will plug into the "main" event in the thread,
        // so 'single' is turned into 'first' and 'last' into 'middle'
        if (this.filledEvents.length === 1) return 'first'
        if (index === 0) return 'first'
        if (index === this.filledEvents.length - 1) return 'middle'
        return 'middle'
      }
    },

    addEvent(event) {
      this.$emit('add-event', event)
    },

    resize(event) {
      this.resizing = !this.resizing
      this.$emit('resized')
    }
  }
}
</script>

<style lang='scss' scoped>
</style>
