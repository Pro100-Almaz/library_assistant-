<template>
    <div class="chat-conversation">
        <div class="chat-conversation__messages">
            <ChatMessage v-for="(message, index) in messages" :key="index" :text="message.text" :author="message.author"/>
        </div>
        <ChatLoading v-if="isLoading" />
    </div>
</template>

<script>
import ChatMessage from '@components/chat/ChatMessage.vue';
import ChatLoading from '@components/chat/ChatLoading.vue';

export default {
  name: 'ChatConversation',
  components: {
    ChatMessage,
    ChatLoading
  },
  data() {
    return {
      audioUrl: 'http://54.162.246.131/app/speech.mp3'
    };
  },
  methods: {
    playAudio() {
      console.log("I have requested audio")
      const player = this.$refs.audioPlayer;
      player.play().catch(error => {
        console.error("Error playing the audio:", error);
        alert('Failed to play the audio.');
      });
    }
  },
  props: {
    messages: {
      type: Array,
      default: () => []
    },
    isLoading: {
      type: Boolean,
      default: false
    }
  }
};
</script>

<style lang="scss" scoped>
    .chat-conversation {
        display: flex;
        padding: 15px;
        flex-direction: column;
        flex-grow: 1;
        overflow: auto;

        .chat-conversation__messages {
            display: flex;
            margin-top: auto;
            flex-direction: column;

            :deep(.chat-message) {
                &:last-child {
                    margin-bottom: 0;
                }
            }
        }
    }
</style>