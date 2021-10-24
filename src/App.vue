<template>
  <div id="app">
    <v-app>
      <v-app-bar app height="60" color="blue" dense dark>
        <div class="mr-4">
          <v-img src="./assets/poland.png" width="50"> </v-img>
        </div>
        <v-toolbar-title>VAT Calculator</v-toolbar-title>
      </v-app-bar>
      <v-main>
        <router-view></router-view>
      </v-main>
    </v-app>
  </div>
</template>