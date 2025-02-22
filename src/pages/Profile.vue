<template>
  <q-page>
    <BaseUserCard v-if='$route.params.pubkey' :pubkey='$route.params.pubkey' class='user-card-header q-my-sm' :header-mode='true'/>
    <q-tabs
      v-model="tab"
      dense
      outline
      align="left"
      active-color='accent'
      :breakpoint="0"
    >
      <q-tab name="posts" label='posts' />
      <q-tab name="following" label='following' />
    </q-tabs>
    <q-tab-panels v-model="tab" animated>
      <q-tab-panel name="posts" class='no-padding'>
        <div>
          <BasePostThread v-for="thread in threads" :key="thread[0].id" :events="thread" @add-event='addEvent'/>
        </div>
      </q-tab-panel>

      <q-tab-panel name="following" class='no-padding'>
        <div v-if="!$store.getters.contacts($route.params.pubkey)">not following anyone</div>
        <div v-else class="flex column relative">
          <q-btn
            v-if="$store.getters.hasMoreContacts($route.params.pubkey)"
            :name="showAllContacts ? 'show less' : 'show all'"
            :label="showAllContacts ? 'show less' : 'show all'"
            :icon-right="showAllContacts ? 'expand_less' : 'expand_more'"
            color="secondary"
            class='q-ma-sm'
            outline
            size='sm'
            @click="showAllContacts = !showAllContacts"
          />
          <div class='q-pl-sm'>
            <BaseUserCard
              v-for="(user) in $store.getters.contacts(
                $route.params.pubkey,
                !showAllContacts
              )"
              :key="user.pubkey"
              :pubkey="user.pubkey"
            />
          </div>
          <q-btn
            v-if='!showAllContacts && $store.getters.hasMoreContacts($route.params.pubkey)'
            icon='more_vert'
            size='xl'
            class='q-pa-md justify-start items-start'
            flat
            dense
            @click="showAllContacts = true"
          />
        </div>
      </q-tab-panel>
    </q-tab-panels>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import {pool} from '../pool'
import helpersMixin from '../utils/mixin'
import {addToThread} from '../utils/threads'
import BaseUserCard from 'components/BaseUserCard.vue'

export default defineComponent({
  name: 'Profile',
  mixins: [helpersMixin],

  components: {
    BaseUserCard,
  },

  data() {
    return {
      threads: [],
      eventsSet: new Set(),
      sub: null,
      showAllContacts: false,
      tab: 'posts'
    }
  },

  activated() {
    this.start()
  },

  deactivated() {
    if (this.sub) this.sub.unsub()
  },

  methods: {
    start() {
      if (this.$route.params.pubkey.toLowerCase().match(/^[0-9a-f]{64}$/)) {
        // ok, it's a pubkey, the default cause
      } else if (
        this.$route.params.pubkey
          .toLowerCase()
          .match(/^web\+nostr:[0-9a-f]{64}$/)
      ) {
        // it's a web+nostr pubkey link
        this.$router.push('/' + this.$route.params.pubkey.slice(-64))
        return
      } else if (
        this.$route.params.pubkey
          .toLowerCase()
          .match(/^web\+nostr:event:[0-9a-f]{64}$/)
      ) {
        // it's a web+nostr event link
        this.$router.push('/event/' + this.$route.params.pubkey.slice(-64))
        return
      } else {
        // it's something we don't understand
        return
      }

      this.listen()
      this.$store.dispatch('useProfile', {pubkey: this.$route.params.pubkey, request: true})
      this.$store.dispatch('useContacts', {pubkey: this.$route.params.pubkey, request: true})
      this.$store.getters
        .contacts(this.$route.params.pubkey)
        ?.forEach(pubkey => this.$store.dispatch('useProfile', {pubkey}))
    },

    listen() {
      this.threads = []
      this.eventsSet = new Set()

      this.sub = pool.sub(
        {
          filter: [
            {
              authors: [this.$route.params.pubkey],
              kinds: [0, 1, 2]
            }
          ],
          cb: async (event, relay) => {
            switch (event.kind) {
              case 0:
                await this.$store.dispatch('addEvent', {event, relay})
                return

              case 1:
              case 2:
                if (this.eventsSet.has(event.id)) return

                this.interpolateEventMentions(event)
                this.eventsSet.add(event.id)
                addToThread(this.threads, event)
                return
            }
          }
        },
        'profile-browser'
      )
    },

    addEvent(event) {
      if (this.eventsSet.has(event.id)) return

      this.interpolateEventMentions(event)
      this.eventsSet.add(event.id)
      addToThread(this.threads, event)
    }
  }
})
</script>

<style lang='scss' scoped>
.q-tabs {
  border-bottom: 1px solid $accent
}
</style>
