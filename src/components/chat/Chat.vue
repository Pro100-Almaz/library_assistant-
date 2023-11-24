<template>
    <div class="chat mx-auto my-10">
        <ChatHeader />
        <ChatConversation ref="messagesRef" :messages="messages" :isLoading="isLoading" />
        <ChatBox @update:messages="onUpdateMessagesDo" />
    </div>
</template>

<script>
import { ref, reactive, nextTick } from 'vue';
import ChatConversation from '@/components/chat/ChatConversation.vue';
import ChatHeader from '@components/chat/ChatHeader.vue';
import ChatBox from '@components/chat/ChatBox.vue';

export default {
  name: 'ChatBase',
  components: {
    ChatHeader,
    ChatBox,
    ChatConversation
  },
  setup() {
    const isLoading = ref( false );
    const messages = reactive( [
      {
        text: 'Hello, i\'m AI Assistant of Nazarbayev University. How can i help you?',
        author: 'assistant'
      },
    ] );
    const messagesRef = ref( null );

    const onUpdateMessagesDo = ( value ) => {
      sendMessage( value );
    };

    const scrollToBottomMessages = async () => {
      await nextTick();
      const element = messagesRef.value;
       element.$el.scrollTo( { top: element.$el.scrollHeight } );
    };

    const sendMessage = async ( textValue ) => {
      messages.push( {
        text: textValue,
        author: 'user'
      } );

      console.log( 'I am at request part' );

      scrollToBottomMessages();

      const options = {
        method: 'POST',
        // headers: {
        //   'Content-Type': 'application/json',
        //   Authorization: `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`,
        // },
        body: JSON.stringify( {
          // model: 'text-davinci-003',
          // prompt: textValue,
          // temperature: 0.9,
          // max_tokens: 150,
          // top_p: 1.0,
          // frequency_penalty: 0,
          // presence_penalty: 0.6,
          chat_id: '220de137-8e3c-477e-a519-8073fdc29bb3',
          text: textValue
        } ),
      };

      try {
        isLoading.value = true;

        const response = await fetch(
          'http://54.162.246.131:8080/v1/chat',
          options
        );
        const data = await response.json();
        console.log( data.text );

        messages.push( {
          text: data.text,
          author: 'assistant'
        } );

        scrollToBottomMessages();

        isLoading.value = false;
      } catch ( error ) {
        console.error( error );

        isLoading.value = false;
      }
    };

    return {
      isLoading,
      messages,
      messagesRef,
      onUpdateMessagesDo
    };
  }
};
</script>

<style lang="scss" scoped>
.chat {
    display: flex;
    height: calc( 100vh - 5rem - 71px );
    width: 800px;
    border: 1px solid var( --color--gray-lighter );
    background: $light;
    flex-direction: column;
    max-width: 90vw;
    border-radius: 10px;
    transition-property: box-shadow, opacity;

    @include box-shadow($dark);
}
</style>