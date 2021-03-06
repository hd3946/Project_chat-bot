<template>
  <div
    ref="containerMessageDisplay"
    :style="{ background: computedMessagesDisplayBg }"
    class="container-message-display"
    @scroll="updateScrollState"
    @drop.prevent="dropInputImage($event)"
    @dragover.prevent="dragOver($event)"
    @dragleave.prevent="dragOver($event)"
  >
    <div
      v-for="(message, index) in messages"
      :key="index"
      class="message-container"
      :class="
        getParticipantById(message.participantId).id == getMyUserId
          ? 'my-message'
          : 'other-message'
      "
    >
      <!-- 닉네임 -->
      <p
        v-if="getParticipantById(message.participantId).id != getMyUserId"
        class="message-username text-sm"
      >
        {{ getParticipantById(message.participantId).name }}
      </p>
      <!-- DB 사진 -->
      <div v-if="message.viewed == 'photo'" class="w-full h-auto p-2">
        <imageDisplay :imagedata="message" />
      </div>
      <!-- 트위터 -->
      <div v-else-if="message.viewed == 'twit'" class="w-full h-auto p-2">
        <MessagesTwit :imagedata="message" />
      </div>
      <!-- 메시지 -->
      <div
        v-else
        class="message-text"
        :style="{
          background:
            getParticipantById(message.participantId).id != getMyUserId
              ? colors.message.others.bg
              : colors.message.myself.bg,
        }"
      >
        <p class="text-1xl">{{ message.content }}</p>
      </div>
      <!-- 시간 -->
      <div
        class="message-timestamp text-xs"
        :style="{
          'justify-content':
            getParticipantById(message.participantId).id != getMyUserId
              ? 'baseline'
              : 'flex-end',
        }"
      >
        {{ message.timestamp.format("LT") }}
        <v-icon class="w-4 ml-2" name="check" base-class="icon-sent"></v-icon>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";
import MessagesTwit from "./MessagesTwit.vue";
import imageDisplay from "./MessagesImage.vue";
export default {
  name: "MessageDisplay",
  components: {
    MessagesTwit,
    imageDisplay,
  },
  data() {
    return {
      updateScroll: false,
      lastMessage: null,
      loading: false,
      messagesDisplayBg: "#f7f3f3",
    };
  },
  props: {
    colors: {
      type: Object,
      required: true,
    },
  },
  computed: {
    ...mapGetters([
      "getParticipantById",
      "messages",
      "getparticipants",
      "getMyUserId",
    ]),
    computedMessagesDisplayBg() {
      return this.messagesDisplayBg;
    },
  },
  created() {
    //console.log("message값확인", this.messages);
  },
  mounted() {
    this.goToBottom();
  },
  updated() {
    if (
      (this.messages.length &&
        this.messages[this.messages.length - 1] !== this.lastMessage) ||
      this.updateScroll
    ) {
      this.goToBottom();
      this.lastMessage = this.messages[this.messages.length - 1];
    }
  },
  methods: {
    ...mapMutations(["fileUploadToSever"]),
    updateScrollState({ target: { scrollTop, clientHeight, scrollHeight } }) {
      if (scrollTop + clientHeight >= scrollHeight) {
        this.updateScroll = true;
      } else {
        this.updateScroll = false;
      }
    },
    goToBottom() {
      let scrollDiv = this.$refs.containerMessageDisplay;
      scrollDiv.scrollTop = scrollDiv.scrollHeight;
      this.updateScroll = false;
    },
    dragOver(e) {
      if (e.type === "dragover") {
        this.messagesDisplayBg = "pink";
        e.target.style.opacity = 0.5;
      } else {
        this.messagesDisplayBg = "#f7f3f3";
        e.target.style.opacity = 1;
      }
    },
    //이미지 드래그 업로드
    dropInputImage(event) {
      let file = Array.from(event.dataTransfer.files, (v) => v)[0];
      console.log(file);
      this.fileUploadToSever(file);
    },
  },
};
</script>

<style scoped>
.container-message-display {
  flex: 1;
  /* overflow-y: hidden; */
  -ms-overflow-style: none;
  overflow-x: hidden;
  display: flex;
  flex-direction: column;
  padding-bottom: 10px;
}
.message-text {
  background: #fff;
  padding: 6px 10px;
  border-radius: 15px;
  margin: 5px 0 5px 0;
  max-width: 70%;
  overflow-wrap: break-word;
  text-align: left;
  white-space: pre-wrap;
}

.message-text > p {
  margin: 5px 0 5px 0;
  height: 100%;
}

.message-timestamp {
  border-radius: 15px;
  max-width: 50%;
  overflow-wrap: break-word;
  text-align: left;
  color: #bdb8b8;
  width: 100%;
  display: flex;
  align-items: center;
}

.my-message > .message-timestamp {
  text-align: right;
}

.my-message {
  justify-content: flex-end;
  padding-right: 15px;
  align-items: flex-end;
}

.other-message {
  justify-content: flex-start;
  padding-left: 15px;
  align-items: flex-start;
}

.other-message > .message-text {
  color: #fff;
  border-bottom-left-radius: 0px;
}

.my-message > .message-text {
  border-bottom-right-radius: 0px;
}
.message-container {
  display: flex;
  flex-wrap: wrap;
  flex-direction: column;
}

.message-username {
  font-weight: bold;
}
.icon-sent {
  color: rgb(129, 127, 127);
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
::-webkit-scrollbar {
  display: none;
}
</style>
