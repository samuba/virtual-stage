<template>
  <div id="app" v-cloak>
    <!-- <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>-->
    <router-view />
  </div>
</template>

<style>
/* purgecss start ignore */
@import "tailwindcss/base";
@import "tailwindcss/components";
/* purgecss end ignore */
@import "tailwindcss/utilities";

html,
body {
  height: 100%;
}

[v-cloak] {
  display: none;
}
</style>
