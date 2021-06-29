<template>
  <q-layout view="lHh Lpr lFf">
    <q-header>
      <q-toolbar class="text-grey-9 bg-white">
        <q-toolbar-title><span class="logo">100</span></q-toolbar-title>
        <q-btn flat round dense icon="more_vert" />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'MainLayout',
});
</script>

<style lang="scss" scoped>
.logo {
  font-weight: bold;
}
</style>
