<script setup>
import { computed, ref } from 'vue'

const goalMinutes = 60
const minutes = ref(7)
const saved = ref(false)
const phraseIndex = ref(0)
const selectedAnswer = ref(null)

const phrases = [
  {
    english: 'I’m looking forward to it.',
    chinese: '我很期待。',
    note: '用来表达对即将发生的事情感到期待。后面接名词或动名词（-ing）。',
    example: 'I’m looking forward to meeting you.',
    exampleChinese: '我很期待见到你。',
  },
  {
    english: 'That makes sense.',
    chinese: '有道理。',
    note: '当你理解并认同对方的解释时，可以这样回应。',
    example: 'That makes sense. Thanks for explaining.',
    exampleChinese: '有道理，谢谢你的解释。',
  },
  {
    english: 'Could you give me a hand?',
    chinese: '你能帮我一下吗？',
    note: '一种自然又礼貌的求助表达，常用于日常对话。',
    example: 'Could you give me a hand with this box?',
    exampleChinese: '你能帮我搬一下这个箱子吗？',
  },
]

const answers = [
  { label: 'A. 感到疲惫的', correct: false },
  { label: 'B. 好奇的', correct: true },
  { label: 'C. 非常忙碌的', correct: false },
  { label: 'D. 容易生气的', correct: false },
]

const currentPhrase = computed(() => phrases[phraseIndex.value])
const completedMinutes = computed(() => Math.min(minutes.value, goalMinutes))
const progressPercent = computed(() => Math.round((completedMinutes.value / goalMinutes) * 100))
const remainingMinutes = computed(() => Math.max(goalMinutes - completedMinutes.value, 0))
const today = new Intl.DateTimeFormat('zh-CN', {
  year: 'numeric',
  month: 'long',
  day: 'numeric',
  weekday: 'long',
}).format(new Date())

function speak(text) {
  if (!('speechSynthesis' in window)) return
  window.speechSynthesis.cancel()
  const utterance = new SpeechSynthesisUtterance(text)
  utterance.lang = 'en-US'
  utterance.rate = 0.88
  window.speechSynthesis.speak(utterance)
}

function chooseAnswer(index) {
  if (selectedAnswer.value !== null) return
  selectedAnswer.value = index
  minutes.value += 3
}

function nextPhrase() {
  phraseIndex.value = (phraseIndex.value + 1) % phrases.length
}
</script>

<template>
  <div class="shell">
    <header>
      <div class="brand">
        <div class="brand-mark">a</div>
        <div>Daily English<small>每天进步一点点</small></div>
      </div>
      <div class="top-right">
        <div class="streak">🔥 <span>7</span> 天连续</div>
        <div class="avatar">E</div>
      </div>
    </header>

    <main>
      <div class="welcome">
        <div>
          <div class="eyebrow">YOUR DAILY PRACTICE</div>
          <h1>嗨～让我们开始今天的共同进步吧</h1>
          <p>每天 60 分钟，让英语慢慢成为你的习惯。</p>
        </div>
        <div class="date">{{ today }}</div>
      </div>

      <div class="layout">
        <div class="main-col">
          <section class="card hero" aria-label="今日单词">
            <div class="hero-top">
              <span class="tag">✦ 今日单词 · WORD OF THE DAY</span>
              <span class="level">B1 · 中级</span>
            </div>
            <div class="word">curious</div>
            <div class="pronounce">/ˈkjʊriəs/ &nbsp; adj.</div>
            <div class="meaning">好奇的；求知欲强的</div>
            <div class="word-actions">
              <button
                class="icon-btn"
                :aria-label="saved ? '已收藏' : '收藏单词'"
                :style="{ color: saved ? '#ffd49a' : '' }"
                title="收藏单词"
                @click="saved = !saved"
              >{{ saved ? '♥' : '♡' }}</button>
              <button class="icon-btn" aria-label="听单词发音" title="听发音" @click="speak('curious')">🔊</button>
            </div>
          </section>

          <section class="card word-example">
            <div>
              <p>She’s <b>curious</b> about how the machine works.</p>
              <div class="translation">她很好奇这台机器是如何运作的。</div>
            </div>
            <button class="listen" aria-label="播放例句" @click="speak('She’s curious about how the machine works.')">▶</button>
          </section>

          <section class="card lesson-card">
            <div class="section-head">
              <h2>今日实用表达</h2>
              <span>{{ String(phraseIndex + 1).padStart(2, '0') }} / 03</span>
            </div>
            <div class="phrase">{{ currentPhrase.english }}</div>
            <p class="phrase-cn">{{ currentPhrase.chinese }}</p>
            <div class="phrase-note">
              <b>用法提示</b> &nbsp;{{ currentPhrase.note }}<br />
              例：{{ currentPhrase.example }}（{{ currentPhrase.exampleChinese }}）
            </div>
            <div class="phrase-actions" style="margin-top: 13px">
              <button class="small-action" @click="speak(currentPhrase.english)">🔊 听发音</button>
              <button class="small-action" @click="nextPhrase">换一个表达 ↗</button>
            </div>
          </section>

          <section class="card quiz">
            <div class="section-head">
              <h2>快速小测</h2>
              <span>单题 · 约 30 秒</span>
            </div>
            <div class="quiz-q">“curious” 最接近下面哪个意思？</div>
            <div class="options">
              <button
                v-for="(answer, index) in answers"
                :key="answer.label"
                class="option"
                :class="{
                  correct: selectedAnswer !== null && answer.correct,
                  wrong: selectedAnswer === index && !answer.correct,
                }"
                :disabled="selectedAnswer !== null"
                @click="chooseAnswer(index)"
              >{{ answer.label }}</button>
            </div>
            <div class="feedback" :class="selectedAnswer === null ? '' : answers[selectedAnswer].correct ? 'good' : 'bad'" aria-live="polite">
              {{ selectedAnswer === null ? '' : answers[selectedAnswer].correct ? '答对了！curious 表示“好奇的”，常用来形容想了解新事物的人。' : '再记一下：curious 的意思是“好奇的”。' }}
            </div>
          </section>
        </div>

        <aside class="side-col">
          <section class="card side-card">
            <div class="section-head">
              <h2>今日学习进度</h2>
              <span>目标 60 分钟</span>
            </div>
            <div class="progress-row">
              <div>
                <div class="progress-number">
                  <span>{{ completedMinutes }}</span>
                  <small style="font-size: 14px; color: #7f8981; font-weight: 500"> / 60 分钟</small>
                </div>
                <div class="progress-label">保持节奏，你做得很好</div>
              </div>
              <div class="progress-ring" :style="{ background: `conic-gradient(var(--green) 0 ${progressPercent}%, #e9eee8 ${progressPercent}% 100%)` }">
                <span>{{ progressPercent }}%</span>
              </div>
            </div>
            <div class="progress-bar"><i :style="{ width: `${progressPercent}%` }"></i></div>
            <div class="progress-caption">
              <span>今日目标</span>
              <span>{{ remainingMinutes === 0 ? '今日目标已完成' : `还差 ${remainingMinutes} 分钟` }}</span>
            </div>
            <div class="daily-list">
              <div class="daily-item"><span class="check">✓</span>学习今日单词 <span class="daily-count">1 / 30</span></div>
              <div class="daily-item"><span class="check">✓</span>阅读实用表达 <span class="daily-count">1 / 10</span></div>
              <div class="daily-item">
                <span class="check" :class="{ todo: selectedAnswer === null }">{{ selectedAnswer === null ? '·' : '✓' }}</span>
                完成快速小测 <span class="daily-count">{{ selectedAnswer === null ? '0 / 1' : '1 / 1' }}</span>
              </div>
            </div>
            <div class="goal">💡 <b>小建议：</b>试着用今天的单词造一个关于自己的句子，记忆会更牢固。</div>
          </section>

          <section class="card week">
            <div class="section-head"><h2>本周打卡</h2><span>7 天连续 🔥</span></div>
            <div class="week-grid">
              <div class="day">一<div class="dot done">✓</div></div>
              <div class="day">二<div class="dot done">✓</div></div>
              <div class="day">三<div class="dot done">✓</div></div>
              <div class="day">四<div class="dot done">✓</div></div>
              <div class="day">五<div class="dot today">●</div></div>
              <div class="day">六<div class="dot">6</div></div>
              <div class="day">日<div class="dot">7</div></div>
            </div>
          </section>
          <div class="motivation">
            <strong>“A little progress each day adds up to big results.”</strong>
            每天的一点点进步，积累起来就会有很大的成果。
          </div>
        </aside>
      </div>
    </main>

    <div class="footer">DAILY ENGLISH · LEARN A LITTLE, EVERY DAY</div>
  </div>
</template>
