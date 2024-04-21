<template>
    <div class="chat mx-auto my-10">
        <ChatHeader />
        <ChatConversation ref="messagesRef" :messages="messages" :isLoading="isLoading" />
        <ChatBox @update:messages="sendMessage" />
    </div>
</template>

<script>
import { ref, reactive, nextTick, onMounted } from 'vue';
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
    const isLoading = ref(false);
    const chat_id = ref(null);
    const messages = reactive([
      {
        text: "Hello, I'm AI Assistant of Nazarbayev University. How can I help you?",
        author: 'assistant'
      },
    ]);

    const messagesRef = ref(null);

    const sendMessage = async (textValue) => {
      messages.push({ text: textValue, author: 'user' });
      console.log('I am at request part');
      await scrollToBottomMessages();
      
      const options = {
        method: 'POST',
        body: JSON.stringify({
          chat_id: chat_id.value,
          text: textValue
        }),
      };

      isLoading.value = true;
      try {
        const response = await fetch('http://54.162.246.131:8080/v1/chat', options);
        const data = await response.json();
        messages.push({ text: data.text, author: 'assistant' });
        await scrollToBottomMessages();
      } catch (error) {
        console.error(error);
      } finally {
        isLoading.value = false;
      }
    };

    const scrollToBottomMessages = async () => {
      await nextTick();
      messagesRef.value?.$el.scrollTo({ top: messagesRef.value.$el.scrollHeight });
    };

    return {
      isLoading,
      messages,
      messagesRef,
      sendMessage // Updated to expose sendMessage
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