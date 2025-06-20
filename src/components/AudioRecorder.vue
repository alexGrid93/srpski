<template>
  <div class="audio-recorder">
    <a-button size="small" @click="toggleRecording">
      {{ recording ? "⏹ Стоп" : audioURL ? "🔁" : "🎙" }}
    </a-button>

    <a-button
      v-if="audioURL"
      size="small"
      type="default"
      class="play-btn"
      @click="playAudio"
    >
      ▶️
    </a-button>
  </div>
</template>

<script setup lang="ts">
import { ref, defineEmits } from "vue";
import { message } from "ant-design-vue";

const emit = defineEmits<{
  (e: "recorded", data: string): void;
}>();

const recording = ref(false);
const mediaRecorder = ref<MediaRecorder | null>(null);
const chunks: BlobPart[] = [];
const audioURL = ref<string>("");
const player = ref<HTMLAudioElement | null>(null);

const streamRef = ref<MediaStream | null>(null); // <--- добавляем

async function initRecorder() {
  if (!mediaRecorder.value) {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      streamRef.value = stream; // сохраняем для остановки

      mediaRecorder.value = new MediaRecorder(stream);
      mediaRecorder.value.ondataavailable = (e) => chunks.push(e.data);
      mediaRecorder.value.onstop = () => {
        const blob = new Blob(chunks, { type: "audio/webm" });
        const reader = new FileReader();
        reader.onload = () => {
          const base64 = reader.result as string;
          audioURL.value = base64;
          emit("recorded", base64);
        };
        reader.readAsDataURL(blob);
        chunks.length = 0;

        streamRef.value?.getTracks().forEach((track) => track.stop());
        mediaRecorder.value = null;
        streamRef.value = null;
      };
    } catch (err) {
      message.error("Не удалось получить доступ к микрофону");
    }
  }
}

async function toggleRecording() {
  await initRecorder();
  if (!mediaRecorder.value) return;

  if (!recording.value) {
    mediaRecorder.value.start();
    recording.value = true;
  } else {
    mediaRecorder.value.stop();
    recording.value = false;
  }
}

function playAudio() {
  if (!audioURL.value) return;

  if (player.value) {
    player.value.pause();
    player.value.currentTime = 0;
  }

  player.value = new Audio(audioURL.value);
  player.value.play();
}
</script>

<style scoped>
.audio-recorder {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.play-btn {
  padding: 0 0.6rem;
}
</style>
