<template>
  <div class="fixed bottom-8 right-8">
    <button
      @click="toggleChat"
      class="bg-blue-500 hover:bg-blue-600 text-white rounded-full p-3 shadow-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-all duration-300"
      :class="{ 'rotate-180': isChatOpen }"
    >
      <MessageSquare v-if="!isChatOpen" class="w-6 h-6" />
      <X v-else class="w-6 h-6" />
    </button>

    <transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="transform scale-95 opacity-0"
      enter-to-class="transform scale-100 opacity-100"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="transform scale-100 opacity-100"
      leave-to-class="transform scale-95 opacity-0"
    >
      <div
        v-if="isChatOpen"
        class="absolute border bottom-16 right-0 w-80 h-[450px] bg-white rounded-[15px] shadow-xl overflow-hidden"
      >
        <div class="flex flex-col h-full">
          <div class="bg-blue-500 text-white p-3">
            <h2 class="text-lg font-semibold">Aisha AI</h2>
          </div>
          <div
            ref="messageContainer"
            class="flex-1 overflow-y-auto p-3 space-y-4"
          >
            <div v-for="message in messages" :key="message.id" class="flex">
              <div
                :class="[
                  'max-w-2/3 px-2 py-1',
                  message.isUser
                    ? 'bg-[#1F93FF] text-white ml-auto rounded-t-[13px] rounded-bl-[13px]'
                    : 'bg-gray-100 rounded-t-[13px] rounded-br-[13px]',
                ]"
              >
                <span class="text-sm">{{ message.content }}</span>
              </div>
            </div>
            <div v-if="isLoading" class="flex">
              <div
                class="bg-gray-200 rounded-t-[13px] rounded-br-[13px] h-8 w-16 shadow animate-pulse flex items-center justify-center"
              >
                <div class="loader"></div>
              </div>
            </div>
          </div>
          <div class="p-4 bg-white border-t">
            <form @submit.prevent="sendMessage" class="flex space-x-2">
              <input
                v-model="userInput"
                type="text"
                placeholder="Type your message..."
                class="flex-1 p-2 text-sm border rounded-lg transition-300 outline-none focus:outline-blue-500"
              />
              <button
                type="submit"
                class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-300"
                :disabled="isLoading"
              >
                Send
              </button>
            </form>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from "vue";
import { MessageSquare, X } from "lucide-vue-next";

const isChatOpen = ref(false);
const messages = ref([
  {
    id: 1,
    content: "Hello! How can I assist you today?",
    isUser: false,
  },
]);
const userInput = ref("");
const isLoading = ref(false);
const messageContainer = ref(null);

const toggleChat = () => {
  isChatOpen.value = !isChatOpen.value;
};

const scrollToBottom = () => {
  if (messageContainer.value) {
    nextTick(() => {
      messageContainer.value.scrollTop = messageContainer.value.scrollHeight;
    });
  }
};

const sendMessage = async () => {
  if (userInput.value.trim() === "") return;

  messages.value.push({
    id: messages.value.length + 1,
    content: userInput.value,
    isUser: true,
  });

  const userMessage = userInput.value;
  userInput.value = "";
  isLoading.value = true;

  scrollToBottom();

  try {
    const response = await fetch(
      "https://api.sambanova.ai/v1/chat/completions",
      {
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer 3e2529e6-dd6e-4974-b714-bcaa5a0b19fc`,
        },
        method: "POST",
        body: JSON.stringify({
          messages: [
            {
              role: "user",
              content: userMessage,
            },
          ],
          model: "Meta-Llama-3.1-8B-Instruct",
        }),
      }
    );

    if (!response.ok) {
      throw new Error("Network response was not ok");
    }

    const data = await response.json();
    messages.value.push({
      id: messages.value.length + 1,
      content: data.choices[0].message.content,
      isUser: false,
    });

    scrollToBottom();
  } catch (error) {
    console.error("Error fetching response:", error);
    messages.value.push({
      id: messages.value.length + 1,
      content: "Sorry, I couldn't process your request. Please try again.",
      isUser: false,
    });
  } finally {
    isLoading.value = false;
    scrollToBottom();
  }
};

onMounted(() => {
  scrollToBottom();
});
</script>

<style scoped>
/* Target the entire scrollbar */
::-webkit-scrollbar {
  width: 4px; /* Width of the scrollbar */
  height: 12px; /* Height of the scrollbar for horizontal scroll */
}

/* Handle (scroll thumb) */
::-webkit-scrollbar-thumb {
  background: #d0dbf0; /* Color of the scrollbar handle */
  border-radius: 10px; /* Round corners */
}

/* Scrollbar track (background) */
::-webkit-scrollbar-track {
  border-radius: 10px;
}

/* HTML: <div class="loader"></div> */
.loader {
  width: 30px;
  aspect-ratio: 4;
  --_g: no-repeat radial-gradient(circle closest-side, #967676 90%, #0000);
  background: var(--_g) 0% 50%, var(--_g) 50% 50%, var(--_g) 100% 50%;
  background-size: calc(100% / 3) 100%;
  animation: l7 1s infinite linear;
}
@keyframes l7 {
  33% {
    background-size: calc(100% / 3) 0%, calc(100% / 3) 100%, calc(100% / 3) 100%;
  }
  50% {
    background-size: calc(100% / 3) 100%, calc(100% / 3) 0%, calc(100% / 3) 100%;
  }
  66% {
    background-size: calc(100% / 3) 100%, calc(100% / 3) 100%, calc(100% / 3) 0%;
  }
}
</style>
