<template>
  <div
    v-if="page.syncing"
    class="w-100 pa-3 d-flex flex-row justify-center align-center"
  >
    <v-progress-circular class="mr-2" indeterminate size="32" />
    <span>{{ $t('syncingInProgress') }}</span>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapStores } from 'pinia';
import { pageStore } from '~/store';

export default Vue.extend({
  computed: {
    ...mapStores(pageStore)
  }
});
</script>
