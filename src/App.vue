<script setup lang="ts">
import { ref, onMounted, watch, computed } from "vue";
import defaultWords from "./assets/defaultWords.json";
import AudioRecorder from "./components/AudioRecorder.vue";

interface Entry {
  id: string;
  word: string;
  translation: string;
  tip: string;
  pronunciation: string;
  training: string;
}

const STORAGE_KEY = "word_table_entries";
const entries = ref<Entry[]>([]);

const columns = [
  { title: "Слово", dataIndex: "word" },
  { title: "Перевод", dataIndex: "translation" },
  { title: "Примечание", dataIndex: "tip" },
  { title: "Произношение", dataIndex: "pronunciation" },
  { title: "Тренировка", dataIndex: "training" },
  { title: "—", dataIndex: "actions" },
];

const localStorageUsage = computed(() => {
  const value = localStorage.getItem(STORAGE_KEY) || "";
  const bytes = new Blob([value]).size;
  const maxBytes = 5 * 1024 * 1024; // 5MB
  const usedMB = (bytes / 1024 / 1024).toFixed(2);
  const maxMB = (maxBytes / 1024 / 1024).toFixed(0);
  return `${usedMB} МБ из ${maxMB} МБ`;
});

onMounted(() => {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    try {
      entries.value = JSON.parse(saved);
    } catch (e) {
      console.error("Ошибка парсинга сохранённых данных", e);
    }
  } else {
    entries.value = defaultWords as Entry[];
    localStorage.setItem(STORAGE_KEY, JSON.stringify(defaultWords));
  }
});

watch(
  entries,
  (newEntries) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(newEntries));
  },
  { deep: true }
);

function addRow() {
  entries.value.push(createEmptyRow());
}

function addRowToStart() {
  entries.value.unshift(createEmptyRow());
}

function createEmptyRow(): Entry {
  return {
    id: Date.now().toString(),
    word: "",
    translation: "",
    tip: "",
    pronunciation: "",
    training: "",
  };
}

function removeRow(id: string) {
  entries.value = entries.value.filter((entry) => entry.id !== id);
}

function playSound(src: string) {
  const audio = new Audio(src);
  audio.play();
}
</script>

<template>
  <div class="container">
    <h1 class="title">Базовый сербский 🇷🇸</h1>

    <div class="storage-info">Объём данных: {{ localStorageUsage }}</div>

    <a-table
      :columns="columns"
      :data-source="entries"
      :row-key="(record) => record.id"
      bordered
    >
      <template #bodyCell="{ column, record, index }">
        <template v-if="column.dataIndex === 'word'">
          <a-textarea
            v-model:value="record.word"
            placeholder="apple"
            :auto-size="{ minRows: 1, maxRows: 3 }"
          />
        </template>
        <template v-else-if="column.dataIndex === 'translation'">
          <a-textarea
            v-model:value="record.translation"
            placeholder="яблоко"
            :auto-size="{ minRows: 1, maxRows: 3 }"
          />
        </template>
        <template v-else-if="column.dataIndex === 'tip'">
          <a-textarea
            v-model:value="record.tip"
            placeholder="I eat an apple."
            :auto-size="{ minRows: 1, maxRows: 12 }"
          />
        </template>
        <template v-else-if="column.dataIndex === 'pronunciation'">
          <div class="pronunciation-cell">
            <AudioRecorder
              v-if="false"
              :key="record.id + (record.pronunciation || '')"
              @recorded="(data) => (record.pronunciation = data)"
            />
            <a-button
              v-if="record.pronunciation"
              size="small"
              type="default"
              class="play-btn"
              @click="playSound(record.pronunciation)"
            >
              ▶️
            </a-button>
          </div>
        </template>
        <template v-else-if="column.dataIndex === 'training'">
          <div class="pronunciation-cell">
            <AudioRecorder
              :key="record.id + (record.training || '') + '-training'"
              @recorded="(data) => (record.training = data)"
            />
            <a-button
              v-if="record.training"
              size="small"
              type="default"
              class="play-btn"
              @click="playSound(record.training)"
            >
              ▶️
            </a-button>
          </div>
        </template>
        <template v-else-if="column.dataIndex === 'actions'">
          <a-popconfirm
            title="Удалить эту строку?"
            @confirm="removeRow(record.id)"
            ok-text="Да"
            cancel-text="Нет"
          >
            <a-button danger size="small">❌</a-button>
          </a-popconfirm>
        </template>
      </template>
    </a-table>

    <div class="actions">
      <a-button type="primary" @click="addRow">Добавить строку</a-button>
      <a-button type="dashed" @click="addRowToStart">➕ В начало</a-button>
    </div>
  </div>
</template>

<style scoped>
.container {
  padding: 1.5rem;
  background-color: #fff;
  border-radius: 8px;
}
.title {
  text-align: center;
  margin-bottom: 1rem;
}
.storage-info {
  text-align: right;
  margin-bottom: 0.5rem;
  font-size: 0.875rem;
  color: #888;
}
.pronunciation-cell {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.actions {
  margin-top: 1rem;
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
}
</style>
