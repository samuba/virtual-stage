<template>
  <i class="opacity-75 w-6 h-6 pt-1 ml-2" >
    <svg
      xmlns="http://www.w3.org/2000/svg"
      xmlns:xlink="http://www.w3.org/1999/xlink"
      version="1.1"
      x="0px"
      y="0px"
      viewBox="0 0 1000 1000"
      enable-background="new 0 0 1000 1000"
      xml:space="preserve"
      class="rotate"
    >
      <g>
        <path
          d="M503.4,992.7c-24.9,0-50-1.9-75-5.7c-65.7-10-127.5-32.6-183.7-67.2c-54.2-33.4-101-76.4-138.9-128c-37.9-51.5-65-109-80.7-170.7C8.8,557.2,5.7,491.4,15.7,425.7C25.8,360,48.4,298.2,82.9,242c33.4-54.2,76.4-101,128-138.9c51.5-37.9,109-65,170.7-80.7C445.5,6.2,511.2,3,577,13.1c85.8,13.1,166.7,48.6,233.9,102.6c65.2,52.4,116.4,121.1,148.1,198.6c7.1,17.3-1.2,37.1-18.6,44.2c-17.3,7.1-37.1-1.2-44.2-18.6C839.9,202.1,713.7,102.5,566.7,80.1c-231.5-35.3-448.7,124.3-484,355.8c-35.3,231.5,124.3,448.7,355.8,484c231.5,35.3,448.7-124.3,484-355.8c2.8-18.5,20.1-31.2,38.6-28.4c18.5,2.8,31.2,20.1,28.4,38.6C979.6,640,957,701.8,922.4,758c-33.4,54.2-76.4,101-128,138.8c-51.5,37.9-109,65-170.7,80.7C584.2,987.6,543.9,992.7,503.4,992.7z"
        />
      </g>
    </svg>
  </i>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component
export default class LoadingIcon extends Vue {}
</script>

<style scoped lang="postcss">
.rotate {
  animation: rotation 1.5s infinite linear;
}

@keyframes rotation {
  from {
    -webkit-transform: rotate(0deg);
  }
  to {
    -webkit-transform: rotate(359deg);
  }
}
</style>
