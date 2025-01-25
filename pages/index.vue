<template>
  <div class="fixed bottom-12 right-12">
    <div
      class="absolute -z-10 bg-[#D6EDFF] animate-grow-glow w-12 h-12 border rounded-full"
    ></div>
    <button
      @click="$device.isMobile ? toggleExpand() : toggleChat()"
      class="bg-blue-500 text-white rounded-full w-12 h-12 flex items-center justify-center shadow-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-all duration-300"
      :class="{ 'rotate-180': isChatOpen }"
    >
      <img class="w-6 h-6" v-if="!isChatOpen" src="/logo.svg" alt="" />
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
        class="bottom-16 right-0 w-[400px] h-[500px] bg-white sm:rounded-[15px] shadow-xl overflow-hidden"
        :class="
          isExpanded || fromTelegram
            ? 'fixed top-0 left-0 w-full h-full transition-200'
            : 'absolute'
        "
      >
        <div class="flex flex-col h-full">
          <div class="bg-blue-500 text-white p-3 flex justify-between">
            <div>
              <h2 class="text-lg font-semibold">Aisha AI</h2>
              <p class="text-sm">( Chatbot test rejimida ishlamoqda )</p>
            </div>
            <button
              v-if="!fromTelegram"
              class="flex items-center"
              @click="toggleExpand"
            >
              <Expand v-if="!isExpanded" />
              <svg
                v-else
                aria-hidden="true"
                xmlns="http://www.w3.org/2000/svg"
                width="28"
                height="28"
                fill="none"
                viewBox="0 0 24 24"
              >
                <path
                  stroke="currentColor"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M5 9h4m0 0V5m0 4L4 4m15 5h-4m0 0V5m0 4 5-5M5 15h4m0 0v4m0-4-5 5m15-5h-4m0 0v4m0-4 5 5"
                ></path>
              </svg>
            </button>
          </div>
          <div
            ref="messageContainer"
            class="flex-1 overflow-y-auto p-2 space-y-2"
          >
            <h1 class="text-center text-sm">{{ date }}</h1>
            <div
              v-for="message in messages"
              :key="message.id"
              class="flex items-end gap-1"
              :class="[message.isUser ? 'justify-end' : 'justify-start']"
            >
              <img
                v-if="!message.isUser"
                src="/main-logo.svg"
                alt="Main Logo"
                class="w-7 h-7"
              />
              <button
                v-if="message.isUser"
                @click="textToSpeech(message.content, message.id)"
                class="p-1 bg-blue-500 rounded-md hover:bg-blue-600 transition-200"
              >
                <Volume2
                  v-if="!isLoadingAudio[message.id]"
                  class="text-white w-[18px] h-[18px]"
                />
                <Loader
                  v-else
                  class="text-white w-[18px] h-[18px] animate-spin"
                />
              </button>
              <div
                :class="[
                  'max-w-[70%] break-word-own overflow-hidden whitespace-pre-wrap px-2 py-1',
                  message.isUser
                    ? 'bg-[#1F93FF] text-white rounded-t-[13px] rounded-bl-[13px]'
                    : 'bg-gray-100 rounded-t-[13px] rounded-br-[13px]',
                ]"
              >
                <span class="text-sm">{{ message.content }}</span>
              </div>
              <button
                @click="textToSpeech(message.content, message.id)"
                v-if="!message.isUser"
                class="p-1 bg-blue-500 rounded-md hover:bg-blue-600 transition-200"
              >
                <Volume2
                  v-if="!isLoadingAudio[message.id]"
                  class="text-white w-[18px] h-[18px]"
                />
                <Loader
                  v-else
                  class="text-white w-[18px] h-[18px] animate-spin"
                />
              </button>
            </div>
            <div v-if="isLoading" class="flex items-end gap-1">
              <img src="/main-logo.svg" alt="Main Logo" class="w-7 h-7" />
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
                :readonly="isLoading"
                type="text"
                :placeholder="
                  isLoading ? 'Iltimos kuting...' : 'Xabaringizni yozing...'
                "
                class="flex-1 p-2 text-sm border rounded-lg transition-300 outline-none focus:outline-blue-500"
              />
              <transition name="fade" mode="out-in">
                <button
                  v-if="userInput.trim().length > 0"
                  key="send"
                  type="submit"
                  class="py-2 px-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-300"
                  :disabled="isLoading"
                >
                  <SendHorizonal class="w-5 h-5" />
                </button>
                <button
                  v-else
                  key="mic"
                  type="button"
                  class="py-2 px-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-300"
                >
                  <Mic class="w-5 h-5" />
                </button>
              </transition>
            </form>
          </div>
        </div>
      </div>
    </transition>
    <audio ref="audioPlayer" class="hidden" />
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from "vue";
import {
  X,
  Expand,
  Volume2,
  Loader,
  Mic,
  SendHorizonal,
} from "lucide-vue-next";
const { isMobile } = useDevice();
const route = useRoute();
const date = computed(() => {
  return (
    String(new Date().getDate()).padStart(2, "0") +
    "." +
    String(new Date().getMonth() + 1).padStart(2, "0") +
    "." +
    new Date().getFullYear()
  );
});
const fromTelegram = computed(() => {
  return route.query.fromTelegram ? true : false;
});

const toggleExpand = () => {
  if (isMobile) {
    isChatOpen.value = !isChatOpen.value;
  }
  isExpanded.value = !isExpanded.value;
};

const isChatOpen = ref(fromTelegram.value ? true : false);
const isExpanded = ref(false);
const audio = ref(null);
const audioPlayer = ref(null);
const messages = ref([
  {
    id: 1,
    content: "Assalomu alaykum! Bugun sizga qanday yordam berishim mumkin?",
    isUser: false,
  },
]);
const userInput = ref("");
const isLoading = ref(false);
const messageContainer = ref(null);
const isLoadingAudio = ref({});

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

const textToSpeech = async (text, messageId) => {
  isLoadingAudio.value[messageId] = true;
  try {
    const response = await fetch("https://back.aisha.group/api/v1/tts/post/", {
      method: "POST",
      body: JSON.stringify({
        transcript: text,
        language: "uz",
        model: "jaxongir",
      }),
      headers: {
        "Content-Type": "application/json",
        "x-api-key": "kc4MV8lh.WHwNzvE3s5bj9ssEY584Lo3bI3XQwYCc",
      },
    });

    if (!response.ok) {
      throw new Error("Failed to fetch audio");
    }

    const data = await response.json();
    if (!data.audio_path) {
      throw new Error("No audio path received");
    }

    const audioUrl = `https://back.aisha.group/${data.audio_path}`;

    if (audioPlayer.value) {
      audioPlayer.value.src = audioUrl;
      await audioPlayer.value.play();
    } else {
      console.error("Audio player not found");
    }
  } catch (error) {
    console.error("Error in text-to-speech:", error);
  } finally {
    isLoadingAudio.value[messageId] = false;
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

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
