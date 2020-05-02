<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title class="text-center">
          Human Vs Monster
        </q-toolbar-title>

        <q-btn flat round dense type="a" icon="ion-logo-github" href="https://github.com/M1chaelTran/vue-game" target="_blank" />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>

  </q-layout>
</template>
