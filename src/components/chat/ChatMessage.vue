<template>
    <div 
        class="chat-message"
        :class="{
            'is-user': isUser,
            'is-assistant': isAssistant,
        }">
        <div class="chat-message__avatar">
            <AssistantLogoImage v-if="isAssistant" />
            <Icon icon="material-symbols:check-circle" width="20px" height="20px" v-if="isUser" />
        </div>
        <div class="chat-message__text">
            {{ text }}
            <button v-if="isAssistant" @click="fetchAndUpdateAudio" class="audio-control">
                <span v-if="!isPlaying" class="material-symbols-outlined">play_circle</span>
            </button>
            <audio ref="audioPlayer" :src="currentAudioUrl" @ended="resetAudio"></audio>
        </div>
    </div>
</template>

<script>
import { computed, toRefs } from 'vue';
import { Icon } from '@iconify/vue';
import axios from 'axios';
import AssistantLogoImage from '@/assets/assistant_logo.svg';

export default {
  name: 'ChatMessage',
  components: {
    AssistantLogoImage,
    Icon
  },
  props: {
    text: {
      type: String,
      default: null
    },
    author: {
      type: String,
      default: 'assistant'
    },
    audioUrl: String,
  },
  data() {
    return {
      isPlaying: false,
      baseAudioUrl: 'http://54.162.246.131/app/',
      currentAudioFileName: '',
      currentAudioUrl: ''
    };
  },
  watch: {
    currentAudioFileName(newFileName) {
      this.currentAudioUrl = `${this.baseAudioUrl}${newFileName}`;
      this.playAudio();
    }
  },
  methods: {
    fetchAndUpdateAudio() {
      axios.post('http://54.162.246.131:8080/v1/text-to-audio', { 'text': this.text })
        .then(response => {
          this.currentAudioFileName = response.data.text; // Assuming the backend sends the new file name
        })
        .catch(error => {
          console.error('Error fetching audio:', error);
        });
    },
    playAudio() {
      this.$nextTick(() => {
        let audioPlayer = this.$refs.audioPlayer;
        audioPlayer.load(); // Load the new audio source
        audioPlayer.play().catch(error => {
          console.error('Error playing audio:', error);
          alert('Playback failed, please check the console for more details.');
        });
      });
    },
    resetAudio() {
      this.isPlaying = false;
    }
  },

  setup( props ) {
    const { author } = toRefs( props );

    const isUser = computed( () => {
        return author.value === 'user';
    } );

    const isAssistant = computed( () => {
        return author.value === 'assistant';
    } );


    return {
      isUser,
      isAssistant
    };
  }
};
</script>

<style lang="scss" scoped>
    .chat-message {
        display: inline-flex;
        margin-bottom: 15px;
        align-items: flex-end;

        .chat-message__text {
            padding: 15px;
            display: flex;
            flex-direction: row;
            align-items: center;
            .audio-control {
                background: none;
                border: none;
                cursor: pointer;
                padding: 0;

                .material-symbols-outlined {
                    padding: 0%;
                    margin: 0%;
                    width: 15px;
                }
            }

            .audio-control:focus {
                outline: none;
            }
        }
    
        &.is-user {
            margin-left: auto;

            .chat-message__avatar {
                color: $primary;
                order: 2;
                margin-left: 5px;
            }

            .chat-message__text {
                background: $primary;
                color: $light;
                font-size: 14px;
                letter-spacing: 0.5px;
                border-top-left-radius: 15px;
                border-top-right-radius: 15px;
                border-bottom-left-radius: 15px;
                order: 1;
            }
        }

        &.is-assistant {
            margin-right: auto;

            .chat-message__avatar {
                width: 30px;
                margin-right: 5px;
                border-radius: 100%;
                overflow: hidden;
                flex-shrink: 0;

                svg {
                    width: 30px;
                }
            }

            .chat-message__text {
                background: var( --color--gray-lighter );
                color: $dark;
                font-size: 14px;
                letter-spacing: 0.5px;
                border-top-left-radius: 15px;
                border-top-right-radius: 15px;
                border-bottom-right-radius: 15px;
            }
        }
    }
</style>