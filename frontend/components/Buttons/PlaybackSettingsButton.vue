<template>
  <v-menu
    v-model="menu"
    :close-on-content-click="false"
    :close-on-click="true"
    :transition="'slide-y-transition'"
    top
    :nudge-top="nudgeTop"
    offset-y
    min-width="25em"
    max-width="25em"
    :z-index="500"
    class="menu"
    @input="$emit('input', $event)"
  >
    <!-- eslint-disable-next-line vue/no-template-shadow -->
    <template #activator="{ on: menu, attrs }">
      <v-tooltip top>
        <template #activator="{ on: tooltip }">
          <v-btn
            class="align-self-center active-button"
            icon
            v-bind="attrs"
            v-on="{ ...tooltip, ...menu }"
          >
            <v-icon>mdi-cog</v-icon>
          </v-btn>
        </template>
        <span>{{ $t('playbackSettings') }}</span>
      </v-tooltip>
    </template>
    <v-card>
      <v-list color="transparent">
        <v-list-item disabled>
          <v-row align="center">
            <v-col :cols="4">
              <label>{{ $t('quality') }}</label>
            </v-col>
            <v-col :cols="8">
              <v-select />
            </v-col>
          </v-row>
        </v-list-item>
        <v-list-item>
          <v-row align="center">
            <v-col :cols="4">
              <label>{{ $t('audio') }}</label>
            </v-col>
            <v-col :cols="8">
              <media-stream-selector
                v-if="playbackManager.getCurrentItemAudioTrack"
                :media-streams="playbackManager.getCurrentItemAudioTrack"
                type="Audio"
                :default-stream-index="playbackManager.currentAudioStreamIndex"
                @input="setAudio($event)"
              />
            </v-col>
          </v-row>
        </v-list-item>
        <v-list-item v-show="!$vuetify.breakpoint.smAndUp">
          <v-row align="center">
            <v-col :cols="4">
              <label>{{ $t('subtitles') }}</label>
            </v-col>
            <v-col :cols="8">
              <media-stream-selector
                v-if="playbackManager.getCurrentItemSubtitleTracks"
                :media-streams="playbackManager.getCurrentItemSubtitleTracks"
                type="Subtitle"
                :default-stream-index="
                  playbackManager.currentSubtitleStreamIndex
                "
                @input="setSubtitle($event)"
              />
            </v-col>
          </v-row>
        </v-list-item>
        <v-list-item disabled>
          <v-row align="center">
            <v-col :cols="4">
              <label>{{ $t('speed') }}</label>
            </v-col>
            <v-col :cols="8">
              <v-select />
            </v-col>
          </v-row>
        </v-list-item>
        <v-list-item>
          <v-row align="center">
            <v-col :cols="4">
              <label>{{ $t('stretch') }}</label>
            </v-col>
            <v-col :cols="8">
              <v-switch v-model="stretch" />
            </v-col>
          </v-row>
        </v-list-item>
      </v-list>
    </v-card>
  </v-menu>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapStores } from 'pinia';
import { playbackManagerStore } from '~/store';

export default Vue.extend({
  props: {
    nudgeTop: {
      type: [Number, String],
      default: 0
    },
    stretchProp: Boolean
  },
  data() {
    return {
      menu: false,
      stretch: this.stretchProp
    };
  },
  computed: {
    ...mapStores(playbackManagerStore)
  },
  watch: {
    stretchProp(value) {
      this.stretch = value;
    },
    stretch(value) {
      this.$emit('stretch', value);
    }
  },
  methods: {
    setAudio(audioStreamIndex: number) {
      this.playbackManager.currentAudioStreamIndex = audioStreamIndex;
    },
    setSubtitle(subtitleStreamIndex: number) {
      this.playbackManager.currentSubtitleStreamIndex = subtitleStreamIndex;
    }
  }
});
</script>
