<template>
  <div class="flex justify-center p-4 pt-12">
    <div style="min-width: 200px; max-width: 660px;">
      <h1 class="text-xl">
        Du bist dabei die virtuelle Veranstaltung <b>Citychurch Gottesdienst</b> zu betreten.
      </h1>
      
      <!-- enter username -->
      <div v-if="!usernameSet">
        <div class="mt-8 text-lg">Wie ist dein Name?</div>
        <input
          ref="usernameInput"
          class="block appearance-none border rounded w-64 py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        />
        <button
          @click="setUsername($refs.usernameInput.value)"
          class="mt-8 button"
        >
          <span>Weiter</span>
        </button>
      </div>

      <!-- enable webcam -->
      <div v-if="usernameSet">
        <div
          class="mt-8 bg-blue-100 border-t border-b border-blue-500 text-blue-700 px-4 py-3"
          role="alert"
        >
          <p class="font-bold">{{username}} bitte aktiviere deine Webcam</p>
          <p
            class="text-sm"
          >Damit virtuelle Veranstaltungen gut funktionieren ist es wichtig das man sich sehen kann.</p>
          <p
            class="text-sm"
          >Keine Angst es wird nur alle 5 Sekunden ein Bild übetragen und der Ton deines Mikrofons gar nicht.</p>
        </div>

        <div class="flex justify-center mt-8">
          <div class="flex justify-center flex-col">
            <button
              v-if="!showCameraVideo"
              @click="enableCamera"
              class="button"
              :disabled="loadingCamera"
            >
              <span>Kamera einschalten</span>
              <ion-icon v-if="!loadingCamera" name="camera-outline" size="large" class="ml-2 rotate"></ion-icon>
              <loading-icon v-if="loadingCamera" />
            </button>

            <video
              :class="{hide: !showCameraVideo}"
              ref="video"
              autoplay
              height="200"
              preload="auto"
              style="height: 200px;"
            ></video>

            <button
              v-if="showCameraVideo"
              @click="enterRoom"
              class="button mt-8"
            >
              <span>{{enterEventText}}</span>
              <ion-icon size="large" name="enter-outline" class="ml-2"></ion-icon>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import LoadingIcon from '../components/LoadingIcon.vue';

@Component({components: {
  LoadingIcon
}})
export default class Home extends Vue {
  showCameraVideo = false;
  enterEventText = "Veranstaltung betreten";
  username = localStorage.getItem('username');
  loadingCamera = false;

  get usernameSet() {
    return this.username && this.username.trim() != "";
  }

  setUsername(username: string) {
    this.username = username.trim();
    localStorage.setItem('username', this.username);
  }

  enableCamera() {
    if (navigator.mediaDevices) {
      this.loadingCamera = true;
      navigator.mediaDevices
        .getUserMedia({ video: true })
        .then(stream => {
          (this.$refs.video as any).srcObject = stream;
        })
        .catch(err => {
          alert("Error accessing camera:\n" + err.name + "\n" + err.message);
          this.enterEventText = "Veranstaltung ohne Kamera betreten";
        })
        .finally(() => {
          this.showCameraVideo = true;
          this.loadingCamera = false;
        });
    } else {
      alert("Your browser is too old. Please install Chrome or Firefox");
    }
  }

  enterRoom() {
    this.$router.push({ name: "Room"})
  }
}
</script>
<style scoped lang="postcss">
.hide {
  display: none;
}

.button {
  @apply bg-purple-400 text-purple-900 text-xl font-bold py-2 px-4 rounded inline-flex;
}
</style>
