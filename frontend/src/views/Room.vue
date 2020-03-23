<template>
  <div style="background: #191413; color: white; margin: 0 0 0 0; height: 100%;">
    <div class="flex flex-wrap lg:flex-no-wrap">
      <div class="stream-container">
        <iframe
          src="https://www.youtube.com/embed/M6Iig0iQUXo"
          frameborder="0"
          allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
          allowfullscreen
        ></iframe>
      </div>

      <div class="w-full lg:text-base text-4xl">
        <div class="pl-4 py-2 bg-gray-900">
          {{room}}
          <button @click="showChat=!showChat" title="toggle chat" class="ml-3">
            <ion-icon name="chatbox-outline"></ion-icon>
            <div
              v-if="unreadMessages"
              class="bg-red-500 w-2 h-2 rounded-full relative inline-block"
              style="left: -0.5rem; top: -0.38rem;"
            ></div>
          </button>
        </div>

        <div class="flex-1 bg-gray-100" :class="{hide: !showChat}">
          <div class="overflow-auto p-2 text-black" id="chat-messages" style="height: 361px;">
            <div v-for="entry in chat" :key="entry.createdAt" :title="entry.createdAt" class="mt-1">
              <img :src="pictureForUser(entry.authorName)" class="inline h-8" />
              <span :style="'color:' + colorFromString(entry.authorName)">{{entry.authorName}}:</span>
              {{entry.message}}
            </div>
          </div>
          <div class="flex">
            <input
              v-model="currentMessage"
              @keyup.enter="postMessage"
              placeholder="type your message here..."
              class="w-full lg:p-2 p-5 text-black"
            />
            <button
              @click="postMessage"
              title="send message"
              class="bg-white px-4 lg:text-xl text-5xl"
            >
              <ion-icon name="send-outline" class="text-gray-800"></ion-icon>
            </button>
          </div>
        </div>
      </div>
    </div>

    <div id="usersCamContainer" class="pt-1 pl-1"></div>
    <video
      autoplay
      height="200"
      preload="auto"
      class="pic"
      style="opacity: 0; position: absolute; height:200px;"
    ></video>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import LoadingIcon from "../components/LoadingIcon.vue";

interface Chat {
  createdAt: string;
}

@Component({
  components: {
    LoadingIcon
  }
})
export default class Room extends Vue {
  room = "";
  username = "";
  currentMessage = "";
  chat: Chat[] = [] as any;
  showChat = true;
  lastMessageReadAt = "";

  get unreadMessages() {
    if (this.chat.length === 0) return false;
    if (!this.lastMessageReadAt) return true;
    return this.chat[this.chat.length - 1].createdAt != this.lastMessageReadAt;
  }

  get serverBasePath() {
    return window.location.hostname == "localhost" ? "http://localhost:8088" : "";
  }

  mounted() {
    this.switchOnWebcam();
    this.room = this.joinRoom();
    this.username = this.getUsername();

    this.updateRoom();
    this.sendCurrentPic();
    setInterval(() => this.sendCurrentPic(), 5000);
    setInterval(() => this.updateRoom(), 3000);
  }
  
  usersCamContainerElement() {
    return document.getElementById("usersCamContainer")!;
  }

  postMessage() {
    if (!this.currentMessage.trim()) return;
    fetch(this.serverBasePath + "/rooms/" + this.room + "/" + this.username + "/chat", {
      method: "put",
      headers: { "Content-Type": "text/plain; charset=UTF-8" },
      body: this.currentMessage
    })
      .then(res => res.json())
      .then(chat => {
        this.updateChat(chat);
        this.currentMessage = "";
      });
  }

  updateChat(newChat: Chat[]) {
    if (this.showChat && newChat.length > 0) {
      this.lastMessageReadAt = newChat[newChat.length - 1].createdAt;
    }
    this.chat = newChat;
    this.$nextTick(() => {
      const chatElement = this.$el.querySelector("#chat-messages")!;
      chatElement.scrollTop = chatElement.scrollHeight;
    });
  }

  getUsername(): string {
    let name = localStorage.getItem("username");
    if (name) return name;

    name = prompt("Whats your name?");
    console.log("entered username:", name);
    name = name || this.getUsername();
    localStorage.setItem("username", name);
    return name;
  }

  colorFromString(str: string) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      hash = str.charCodeAt(i) + ((hash << 5) - hash);
    }
    return "hsl(" + (hash % 360) + ",100%,30%)";
  }

  joinRoom(): string {
    return "theroom"
    // const roomUrlPrefix = "#room/";
    // if (
    //   window.location.hash &&
    //   window.location.hash.startsWith(roomUrlPrefix)
    // ) {
    //   return window.location.hash.replace(roomUrlPrefix, "");
    // }
    // let roomName = prompt(
    //   "Enter a name for your new room",
    //   this.uuid().substr(0, 13)
    // );
    // roomName = roomName || this.joinRoom();
    // window.location.hash = roomUrlPrefix + roomName;
    // return roomName;
  }

  switchOnWebcam() {
    const videoElement = this.$el.querySelector("video")!;
    if (navigator.mediaDevices) {
      navigator.mediaDevices
        .getUserMedia({ video: true })
        .then(stream => (videoElement.srcObject = stream))
        .catch(
          err =>
            (document.body.innerHTML =
              '<h2 style="color: white">Error accessing camera:<br>' +
              err.name +
              "<br>" +
              err.message +
              "</h2>")
        );
    } else {
      document.body.innerHTML =
        '<h2 style="color: white">Your browser is too old. Please install Chrome or Firefox</h2>';
    }
  }

  updateRoom() {
    fetch(this.serverBasePath +"/rooms/" + this.room)
      .then(res => res.json())
      .then(room => {
        this.updateChat(room.chat);

        this.showPic(this.username, room.users[this.username]); // show own image first

        const images = this.usersCamContainerElement().childNodes;
        images.forEach((image: any) => {
          if (room.users[image.id]) {
            this.showPic(image.id, room.users[image.id].imageData);
            delete room.users[image.id];
          } else {
            image.outerHTML = ""; // remove element
          }
        });

        for (const user in room.users) {
          this.showPic(user, room.users[user].imageData);
        }
      });
  }

  pictureForUser(username: string) {
    const img = this.$el.querySelector("#" + username) as HTMLImageElement;
    return img ? img.src : "";
  }

  showPic(id: string, imageData: string) {
    let img = document.getElementById(id) as HTMLImageElement;
    if (!img) {
      img = document.createElement("img");
      img.setAttribute("id", id);
      img.setAttribute("class", "pic");
      this.usersCamContainerElement().appendChild(img);
    }
    img.src = imageData;
  }

  uuid() {
    return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function(c) {
      const r = (Math.random() * 16) | 0,
        v = c == "x" ? r : (r & 0x3) | 0x8;
      return v.toString(16);
    });
  }

  sendCurrentPic() {
    fetch(this.serverBasePath +"/rooms/" + this.room + "/" + this.username, {
      method: "put",
      body: JSON.stringify({ imageData: this.takePicture(this.username) }),
      headers: { "Content-Type": "application/json" }
    });
  }

  drawProgress(username: string) {
    const videoElement = this.$el.querySelector("video")!;
    const canvas = document.createElement("canvas");
    canvas.width = videoElement.offsetWidth;
    canvas.height = videoElement.offsetHeight;
    const context = canvas.getContext("2d")!;
    context.beginPath();
    context.strokeStyle = "green";
    context.moveTo(canvas.width - 1, 0);
    context.lineTo(canvas.width - 1, videoElement.offsetHeight);
    context.stroke();
    (document.getElementById(username)! as HTMLImageElement).src = canvas.toDataURL("image/jpeg");
  }

  takePicture(username: string) {
    const videoElement = this.$el.querySelector("video")!;
    const canvas = document.createElement("canvas");
    canvas.width = videoElement.offsetWidth;
    canvas.height = videoElement.offsetHeight;
    const context = canvas.getContext("2d")!;
    context.drawImage(
      videoElement,
      0,
      0,
      videoElement.offsetWidth,
      videoElement.offsetHeight
    );
    context.font = "2em Arial";
    context.textBaseline = "hanging";
    context.shadowOffsetX = 0;
    context.shadowOffsetY = 0;
    context.shadowColor = "rgba(255,255,255,0.5)";
    context.shadowBlur = 6;
    context.fillText(username, 5, 5);
    const result = canvas.toDataURL("image/jpeg");
    if (result.length < 10) {
      throw "Camera did not provide image";
    }
    return result;
  }
}
</script>
<style scoped lang="postcss">
.hide {
  display: none;
}

.stream-container iframe {
  width: 784px;
  height: 441px;
}

@media (max-width: 1024px) {
  /* make stream iframe full width on small devices */
  .stream-container {
    position: relative;
    padding-bottom: 56.25%;
    /* ratio 16x9 */
    height: 0;
    overflow: hidden;
    width: 100%;
    height: auto;
  }

  .stream-container iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
}
</style>
<style lang="postcss">
.pic { /* has to be unscoped css because we add elements to dom by hand */
  display: inline-block;
  height: 200px;
  vertical-align: bottom;
  background-color: black;
  @apply pr-1 pb-1
}
</style>