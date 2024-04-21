<template>
    <div class="chat-box">
        <div class="chat-box__content">
            <input
                v-model="text"
                placeholder="Write a message..."
                type="text"
                :disabled="isRecording"
                @keyup.enter="onClickButtonDo"/>

            <div class="buttons">
              <BaseButton
                  color="primary"
                  iconRight="ri:send-plane-fill" 
                  iconSize="20px"  
                  size="medium" 
                  rounded
                  :disabled="isButtonDisabled || isRecording"
                  @click="onClickButtonDo">
                  Send
              </BaseButton>
              <BaseButton
                  v-if="!isRecording"
                  color="info"
                  iconRight="bi:mic" 
                  iconSize="20px"  
                  size="medium" 
                  rounded
                  @click="ToggleMic">
                  Audio
              </BaseButton>
              <BaseButton
                  v-if="isRecording"
                  color="danger"
                  iconRight="bi:mic" 
                  fullwidth="30px"
                  iconSize="20px"  
                  size="medium" 
                  rounded
                  @click="ToggleMic">
                  Recording
              </BaseButton>
            </div>    
        </div>
    </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import BaseButton from '@components/base/BaseButton.vue';

export default {
  name: 'ChatBox',
  components: {
    BaseButton
  },
  setup( props, content ) {
    const isRecording = ref(false);
    const text = ref( '' );
    const micButton = ref('info')

    let Recognition;
    if ('webkitSpeechRecognition' in window) {
        Recognition = window.webkitSpeechRecognition;
    } else if ('SpeechRecognition' in window) {
        Recognition = window.SpeechRecognition;
    } else {
        console.error('Speech Recognition API is not supported in this browser.');
        return; // Optionally, handle the lack of API support gracefully
    }

    const sr = new Recognition();
    sr.continuous = true;
    sr.interimResults = true;

    const isButtonDisabled = computed( () => {
      return text.value === '';
    } );

    const onClickButtonDo = () => {
      content.emit( 'update:messages', text.value );
      text.value = '';
    };

    onMounted(() => {
      sr.continuous = true
	    sr.interimResults = true

      sr.onstart = () => {
        console.log('SR Started');
        isRecording.value = true;
        micButton.value = 'danger';
      };
      sr.onend = () => {
        console.log('SR Stopped');
        isRecording.value = false;
        micButton.value = 'info';
      };
      sr.onresult = (evt) => {
        const t = Array.from(evt.results)
            .map(result => result[0])
            .map(result => result.transcript)
            .join('')
        
        text.value = t;
        console.log(text)
      }
    }); 

    const ToggleMic = () => {
      if (isRecording.value) {
        sr.stop();
      } else {
        sr.start();
      }
    }

    return {
      text,
      isButtonDisabled,
      onClickButtonDo,
      ToggleMic,
      isRecording
    };
  }
};
</script>

<style lang="scss" scoped>
    .chat-box {
        display: flex;
        width: 100%;
        padding: 15px;

        .chat-box__content {
            display: flex;
            position: relative;
            height: 60px;
            width: 100%;
            background: var( --color--gray-lighter );
            border-radius: 15px;
            gap: 5px;

            input {
                overflow: auto;
                display: flex;
                max-width: 85%;
                padding: 0 100px 0 15px;
                background: transparent;
                font-size: 14px;
                flex-grow: 1;
                outline: none;
            }

            .buttons {
                display: flex;
                flex-direction: row;
                position: absolute;
                top: 50%;
                right: 5px;
                transform: translateY( -50% );
                gap: 5px;
            }

        }
    }
</style>