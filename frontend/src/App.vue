<template>
  <div id="app" v-cloak>
    <!-- <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>-->
    <router-view />
  </div>
</template>

<style>
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";

[v-cloak] {
  display: none;
}
</style>
