<template>
  <div class="inline-block mr-1 mb-1">
    <div class="relative bg-black" style="height:200px;" :style="!hasImage ? 'width: 200px' : ''">
      <template v-if="hasImage">
        <img :src="imageData" />
        <div class="absolute top-0 px-1 text-lg font-medium text-very-dark text-shadow">{{name}}</div>
      </template>
      <template v-else>
        <div
          class="absolute text-2xl font-medium text-gray-100"
          style="top: 50%; left: 50%; transform: translate(-50%, -50%);"
        >{{name}}</div>
      </template>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from "vue-property-decorator";

@Component
export default class AudienceMember extends Vue {
  @Prop({ required: true })
  name!: string

  @Prop({default: ""})
  imageData!: string;

  get hasImage() {
    return this.imageData.length > 50000;
  }
}
</script>

<style scoped lang="postcss">
.text-shadow {
  text-shadow: 0 0px 5px rgba(255, 255, 255, 0.7);
}
</style>
