<template>
  <item-cols>
    <template #left>
      <v-row justify="center" justify-md="start">
        <v-col cols="7" md="3">
          <card :item="item" />
        </v-col>
        <v-col cols="12" md="9">
          <h1
            class="text-h4 font-weight-light"
            :class="{ 'text-center': !$vuetify.breakpoint.mdAndUp }"
          >
            {{ item.Name }}
          </h1>
          <h2
            v-if="item.OriginalTitle && item.OriginalTitle !== item.Name"
            class="text-subtitle-1"
            :class="{ 'text-center': !$vuetify.breakpoint.mdAndUp }"
          >
            {{ item.OriginalTitle }}
          </h2>
          <div
            class="text-caption text-h4 font-weight-medium mt-2"
            :class="{ 'text-center': !$vuetify.breakpoint.mdAndUp }"
          >
            <media-info :item="item" year runtime rating ends-at />
          </div>
          <v-row
            class="my-4 align-center"
            :class="{
              'justify-center': !$vuetify.breakpoint.mdAndUp,
              'ml-0': $vuetify.breakpoint.mdAndUp
            }"
          >
            <play-button class="mr-2" :item="item" />
            <like-button :item="item" class="mr-2" />
            <mark-played-button :item="item" class="mr-2" />
            <item-menu :item="item" />
          </v-row>
          <v-col cols="12" md="10">
            <v-row
              v-if="item && item.GenreItems && item.GenreItems.length > 0"
              align="center"
            >
              <v-col :cols="12" :sm="2" class="px-0 text-truncate">
                <label class="text--secondary">{{ $t('genres') }}</label>
              </v-col>
              <v-col class="px-0" :cols="12" :sm="10">
                <v-slide-group>
                  <v-slide-item
                    v-for="(genre, index) in item.GenreItems"
                    :key="`genre-${genre.Id}`"
                  >
                    <v-chip
                      small
                      link
                      :class="{ 'ml-2': index > 0 }"
                      nuxt
                      :to="`/genre/${genre.Id}?type=${item.Type}`"
                    >
                      {{ genre.Name }}
                    </v-chip>
                  </v-slide-item>
                </v-slide-group>
              </v-col>
            </v-row>
            <v-row
              v-if="
                item && directors.length > 0 && !$vuetify.breakpoint.smAndUp
              "
              align="center"
            >
              <v-col
                :cols="12"
                :sm="2"
                class="mt-sm-3 py-sm-0 px-0 text-truncate"
              >
                <label class="text--secondary">{{ $t('directing') }}</label>
              </v-col>
              <v-col :cols="12" :sm="10">
                <v-row dense>
                  <v-col
                    v-for="director in directors"
                    :key="director.Id"
                    cols="auto"
                  >
                    <v-chip
                      small
                      link
                      nuxt
                      :to="getItemDetailsLink(director, 'Person')"
                    >
                      {{ director.Name }}
                    </v-chip>
                  </v-col>
                </v-row>
              </v-col>
            </v-row>
            <v-row
              v-if="item && writers.length > 0 && !$vuetify.breakpoint.smAndUp"
              align="center"
            >
              <v-col
                :cols="12"
                :sm="2"
                class="mt-sm-3 py-sm-0 px-0 text-truncate"
              >
                <label class="text--secondary">{{ $t('writing') }}</label>
              </v-col>
              <v-col :cols="12" :sm="10">
                <v-row dense>
                  <v-col v-for="writer in writers" :key="writer.Id" cols="auto">
                    <v-chip
                      small
                      link
                      nuxt
                      :to="getItemDetailsLink(writer, 'Person')"
                    >
                      {{ writer.Name }}
                    </v-chip>
                  </v-col>
                </v-row>
              </v-col>
            </v-row>
          </v-col>
          <div>
            <p
              v-if="item.Taglines && item.Taglines.length > 0"
              class="text-subtitle-1 text-truncate"
            >
              {{ item.Taglines[0] }}
            </p>
            <p class="item-overview">{{ item.Overview }}</p>
          </div>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <season-tabs v-if="item.Type === 'Series'" :item="item" />
        </v-col>
      </v-row>
    </template>
    <template #right>
      <div v-if="crew.length > 0">
        <h2 class="text-h6 text-sm-h5">{{ $t('item.crew') }}</h2>
        <person-list :items="crew" />
      </div>
      <div v-if="actors.length > 0">
        <h2 class="text-h6 text-sm-h5">{{ $t('item.cast') }}</h2>
        <person-list :items="actors" />
      </div>
      <related-items :item="item" vertical />
    </template>
  </item-cols>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapStores } from 'pinia';
import { BaseItemDto, BaseItemPerson, ImageType } from '@jellyfin/client-axios';
import { Context } from '@nuxt/types';
import { getBlurhash } from '~/utils/images';
import { getItemDetailsLink, isValidMD5 } from '~/utils/items';
import { authStore, pageStore } from '~/store';

export default Vue.extend({
  meta: {
    backdrop: true,
    transparentAppBar: true
  },
  validate(ctx: Context) {
    return isValidMD5(ctx.route.params.itemId);
  },
  async asyncData({ params, $api }) {
    const auth = authStore();

    const itemId = params.itemId;

    const item = (
      await $api.userLibrary.getItem({
        userId: auth.currentUserId,
        itemId
      })
    ).data;

    return { item };
  },
  data() {
    return {
      item: {} as BaseItemDto,
      backdropImageSource: '',
      currentVideoTrack: undefined as number | undefined,
      currentAudioTrack: undefined as number | undefined,
      currentSubtitleTrack: undefined as number | undefined
    };
  },
  head() {
    return {
      title: this.page.title
    };
  },
  computed: {
    ...mapStores(pageStore),
    crew(): BaseItemPerson[] {
      let crew: BaseItemPerson[] = [];

      if (this.item.People) {
        crew = this.item.People.filter((person: BaseItemPerson) => {
          return ['Director', 'Writer'].includes(person.Type || '');
        });
      }

      return crew;
    },
    actors: {
      get(): BaseItemPerson[] {
        if (this.item.People) {
          return this.item.People.filter((person: BaseItemPerson) => {
            return person.Type === 'Actor';
          }).slice(0, 10);
        } else {
          return [];
        }
      }
    },
    directors: {
      get(): BaseItemPerson[] {
        return this.crew.filter(
          (person: BaseItemPerson) => person.Type === 'Director'
        );
      }
    },
    writers: {
      get(): BaseItemPerson[] {
        return this.crew.filter(
          (person: BaseItemPerson) => person.Type === 'Writer'
        );
      }
    }
  },
  watch: {
    item: {
      handler(val: BaseItemDto): void {
        this.page.title = val.Name || '';

        this.page.backdrop.blurhash = getBlurhash(val, ImageType.Backdrop);
      },
      immediate: true,
      deep: true
    }
  },
  methods: {
    getItemDetailsLink
  }
});
</script>
