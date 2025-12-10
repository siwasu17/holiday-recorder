<template>
  <div id="activity-tracker-app" class="app-container">
    <div class="header-fixed">
      <div class="date-navigation">
        <button @click="previousDay" class="nav-button">&lt; 前の日</button>
        <h2 class="current-date">{{ formattedDate }}</h2>
        <button @click="nextDay" class="nav-button">次の日 &gt;</button>
      </div>
    </div>

    <hr />

    <main class="main-content-scrollable">
      <table class="timetable">
        <thead>
          <tr>
            <th class="time-column-header">時間</th>
            <th class="activity-column-header">活動内容</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="slot in timeSlots"
            :key="slot.start"
            @click="selectTimeSlot(slot.start)"
            :class="['time-slot', { 'is-selected': currentTimeSlot === slot.start }]"
          >
            <td class="time-label">{{ slot.label }}</td>
            <td class="activity-cell">
              <div class="activity-item"></div>
            </td>
          </tr>
        </tbody>
      </table>
    </main>

    <hr />

    <div class="footer-fixed">
      <div class="category-buttons">
        <button
          v-for="category in categories"
          :key="category.key"
          class="category-button"
          @click="selectCategory(category.key)"
        >
          {{ category.label }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, shallowReactive } from 'vue'

const currentDate = ref(new Date())

// TODO: keyはindex番号の方が扱いやすいと思う
const currentTimeSlot = ref<string | null>(null)

type ActivityTasksMap = Map<string, string[]>
// TODO: key-value
// slotKey -> [category, category...]
// Map構造の方が良さそう
const activities: ActivityTasksMap = shallowReactive(new Map())

/**
 * 活動カテゴリのリスト (定数)
 */
const categories = [
  { key: 'work', label: '仕事' },
  { key: 'study', label: '勉強' },
  { key: 'rest', label: '休憩' },
  { key: 'move', label: '移動' },
  { key: 'etc', label: 'その他' },
]

/**
 * 時間割のタイムスロットデータ (定数)
 * 8時から26時までを2時間単位で定義
 */
const timeSlots = [
  { start: '08:00', label: '8:00 - 10:00' },
  { start: '10:00', label: '10:00 - 12:00' },
  { start: '12:00', label: '12:00 - 14:00' },
  { start: '14:00', label: '14:00 - 16:00' },
  { start: '16:00', label: '16:00 - 18:00' },
  { start: '18:00', label: '18:00 - 20:00' },
  { start: '20:00', label: '20:00 - 22:00' },
  { start: '22:00', label: '22:00 - 24:00' },
  { start: '00:00+', label: '24:00 - 26:00' }, // 翌日の0:00-2:00に相当
]

// --- Computed Properties (算出プロパティ) ---

/**
 * 表示日付を整形して返す (computed: 依存する値が変化したら再計算)
 */
const formattedDate = computed(() => {
  return currentDate.value.toLocaleDateString('ja-JP', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    weekday: 'short',
  })
})

// --- Methods (関数) ---

/**
 * 日付を指定日数分変更する
 * @param {number} days - 変更する日数 (+1で翌日、-1で前日)
 */
const changeDay = (days: number) => {
  const newDate = new Date(currentDate.value)
  newDate.setDate(newDate.getDate() + days)
  currentDate.value = newDate
}

/**
 * 「前の日」ボタンの処理
 */
const previousDay = () => {
  changeDay(-1)
}

/**
 * 「次の日」ボタンの処理
 */
const nextDay = () => {
  changeDay(1)
}

/**
 * カテゴリボタンが押された時の処理
 * @param {string} categoryKey - 選択されたカテゴリのキー
 */
const selectCategory = (categoryKey: string) => {
  if (!!currentTimeSlot.value) {
    const key = currentTimeSlot.value
    const categories = activities.get(key)
    if (!!categories) {
      categories.push(categoryKey)
      activities.set(key, categories)
    }
  }
  console.log(activities)
}

const selectTimeSlot = (timeSlotKey: string) => {
  currentTimeSlot.value = timeSlotKey
}
</script>

<style scoped>
/* スタイルは先に提示したものと同じ考え方で実装されます */
.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  overflow: hidden;
}

.header-fixed,
.footer-fixed {
  position: fixed;
  left: 0;
  right: 0;
  z-index: 10;
  background: #fff; /* 背景色を設定してメインコンテンツに重ならないようにする */
  padding: 10px 0;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.header-fixed {
  top: 0;
}

.footer-fixed {
  bottom: 0;
}

.main-content-scrollable {
  flex-grow: 1;
  overflow-y: auto;
  /* ヘッダーとフッターの高さに応じて調整 */
  padding-top: 100px;
  padding-bottom: 80px;
}

.date-navigation {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 10px;
}

.timetable {
  width: 100%;
  border-collapse: collapse;
}

.time-slot {
  height: 80px; /* スロットの視覚的な高さを確保 */
  border-bottom: 1px solid #eee;
}

.time-slot.is-selected {
  background: #007bff;
  color: white;
  border-color: #007bff;
}

.time-label {
  width: 25%;
  text-align: center;
}

.category-buttons {
  display: flex;
  justify-content: space-around;
  padding: 10px;
}

.category-button {
  padding: 8px 12px;
  border: 1px solid #ccc;
  background: #f9f9f9;
  cursor: pointer;
  border-radius: 5px;
}
</style>
