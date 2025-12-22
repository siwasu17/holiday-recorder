<template>
  <div id="activity-tracker-app" class="app-container">
    <div class="header-fixed">
      <div class="date-navigation">
        <button @click="previousDay" class="nav-button">&lt; Prev</button>
        <h2 class="current-date">{{ formattedDate }}</h2>
        <button @click="nextDay" class="nav-button">Next &gt;</button>
      </div>
    </div>

    <hr />

    <main class="main-content-scrollable">
      <table class="timetable">
        <tbody>
          <tr
            v-for="slot in timeSlots"
            :key="slot.start"
            @click="selectTimeSlot(slot.start)"
            :class="['time-slot', { 'is-selected': currentTimeSlot === slot.start }]"
          >
            <td class="time-label">{{ slot.label }}</td>
            <td class="activity-cell">
              <div
                class="activity-item"
                v-for="actKey in activities.get(slot.start)"
                :key="actKey"
                :style="{ backgroundColor: getActColor(actKey) }"
              >
                {{ getActLabel(actKey) }}
              </div>
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
          :style="{ backgroundColor: category.color, borderColor: category.border }"
          @click="selectCategory(category.key)"
        >
          {{ category.label }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, reactive } from 'vue'

const currentDate = ref(new Date())

const currentTimeSlot = ref<string | null>(null)

type ActivityTasksMap = Map<string, string[]>

// timeSlot -> taskCategories
const activities: ActivityTasksMap = reactive(new Map())

/**
 * 活動カテゴリのリスト (定数)
 */
const categories = [
  { key: 'rest', label: '休息', color: '#BDE0FE', border: '#A2D2FF' }, // 水色
  { key: 'meal', label: '食事', color: '#FFECD1', border: '#FFD7BA' }, // 淡いオレンジベージュ
  { key: 'exercise', label: '運動', color: '#CAFFBF', border: '#99E2B4' }, // 薄緑
  { key: 'culture', label: '文化', color: '#FFD6A5', border: '#FFC8DD' }, // 薄橙
  { key: 'dev', label: '開発', color: '#A0C4FF', border: '#9BF6FF' }, // 青
  { key: 'housework', label: '家事(定)', color: '#FDFFB6', border: '#E9EDC9' }, // 薄黄
  { key: 'task', label: '家事(単)', color: '#FFADAD', border: '#FF8B94' }, // 薄赤
  { key: 'plan', label: '検討', color: '#E2E2E2', border: '#CCCCCC' }, // グレー
  { key: 'event', label: '行事', color: '#FFC6FF', border: '#FDBBFF' }, // ピンク
  { key: 'etc', label: 'その他', color: '#FFFFFC', border: '#D8E2DC' }, // オフホワイト
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

const getActLabel = (actKey: string) => {
  return categories.find((c) => c.key == actKey)?.label ?? '不明'
}

const getActColor = (actKey: string) => {
  return categories.find((c) => c.key == actKey)?.color ?? '#000000'
}
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

const selectCategory = (taskCategory: string) => {
  if (!!currentTimeSlot.value) {
    const timeSlot = currentTimeSlot.value
    let currentActivites = activities.get(timeSlot) ?? []

    // 選択した時間枠にタスクカテゴリを追加
    if (!!currentActivites) {
      currentActivites.push(taskCategory)
    } else {
      currentActivites = [taskCategory]
    }
    activities.set(timeSlot, currentActivites)

    if (currentActivites.length >= 4) {
      // 4つ埋まったら次の時間に移動する
      const timeSlotIndex = timeSlots.findIndex((slot) => slot.start === currentTimeSlot.value)
      if (timeSlotIndex + 1 >= timeSlots.length) {
        // 一番下まで来ていたら選択解除
        currentTimeSlot.value = null
      } else {
        // 次のスロットに移動
        currentTimeSlot.value = timeSlots[timeSlotIndex + 1]?.start ?? null
      }
    }
  }
  console.log([...activities])
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
  background: #bcebd2;
}

.time-label {
  width: 25%;
  text-align: center;
}

.category-buttons {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
  padding: 10px;
  gap: 8px;
}

.category-button {
  padding: 8px 12px;
  border: 1px solid #ccc;
  background: #f9f9f9;
  cursor: pointer;
  border-radius: 5px;
  color: #333;
  font-weight: bold;
}

.activity-cell {
  /* セル内の余白を調整（アイテムとの間に少し隙間を作る場合） */
  padding: 4px;
}

.activity-item {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  box-sizing: border-box;

  padding: 2px 8px; /* 上下を極限まで詰め、左右に少し余裕を持たせる */
  font-size: 0.85rem; /* 文字を少し小さく */
  min-height: 24px; /* 最低限の高さだけ確保 */
  margin-bottom: 2px; /* アイテム間の隙間も最小限に */
  border-radius: 4px;

  /* --- 文字が溢れた時の処理（1行に収める） --- */
  white-space: nowrap; /* 折り返し禁止 */
  overflow: hidden; /* はみ出しを隠す */
  text-overflow: ellipsis; /* はみ出た分を「...」にする */

  /* border: 1px solid #dcdcdc; */
}
/* 最後のアイテムの余白を消す */
.activity-item:last-child {
  margin-bottom: 0;
}
</style>
