<script setup>
import { computed } from 'vue'
const props = defineProps({
  games: { type: Array, required: true },
  selectedGame: { type: Object, default: null },
  gamePage: { type: Object, required: true },
  latestDrawIssueLabel: { type: String, default: '--' },
  latestDrawNumbers: { type: Array, default: () => [] },
  latestDrawSpecial: { type: [String, Number], default: '--' },
  latestDrawResultId: { type: [String, Number], default: 'draw' },
  drawResultsError: { type: String, default: '' },
  drawResultsLoading: { type: Boolean, default: false },
  winResultsError: { type: String, default: '' },
  winResultsLoading: { type: Boolean, default: false },
  availablePlayModes: { type: Array, default: () => [] },
  selectedPlayMode: { type: String, default: '' },
  currentPlayModeLabel: { type: String, default: '--' },
  currentIssueStatusText: { type: String, default: '' },
  playSettingsError: { type: String, default: '' },
  playSettingsLoading: { type: Boolean, default: false },
  selectedBetPlay: { type: String, default: '' },
  selectedBetPlays: { type: Array, default: () => [] },
  showTemaSidePlays: { type: Boolean, default: true },
  showTemaSection: { type: Boolean, default: true },
  showBallSelector: { type: Boolean, default: true },
  trendRows: { type: Array, default: () => [] },
  selectedBall: { type: Number, default: 1 },
  ballOptions: { type: Array, default: () => [] },
  pageTitle: { type: String, default: '' },
  formatDateTime: { type: Function, required: true },
  getPlayOdds: { type: Function, required: true },
  getCornerTokens: { type: Function, required: true },
})

const emit = defineEmits(['select-game', 'change-play-mode', 'select-bet-play', 'select-ball'])

const topTriplePlays = ['1念2', '1念3', '1念4']
const leftTriplePlays = ['2念1', '2念4', '2念3']
const rightTriplePlays = ['4念1', '4念2', '4念3']
const bottomTriplePlays = ['3念2', '3念1', '3念4']
const zhongPlayNames = ['234中', '134中', '124中', '123中']
const bigSmallOddEvenPlays = ['大', '小', '单', '双']
const tongPlayNames = [
  '23一通',
  '13二通',
  '12三通',
  '12四通',
  '24二通',
  '14一通',
  '14三通',
  '13四通',
  '34一通',
  '34二通',
  '24三通',
  '23四通',
]
const jiaPlayNames = ['1加23', '1加34', '2加14', '2加34', '3加12', '3加14', '4加12', '4加23']
const temaNumberPlays = Array.from({ length: 20 }, (_, index) => String(index + 1))
const temaSidePlays = ['特大单', '特小单', '特大双', '特小双']
const visibleTemaSidePlays = computed(() => {
  return props.showTemaSidePlays ? temaSidePlays : []
})

const isPlaySelected = (playName) => {
  return props.selectedBetPlays.includes(playName)
}
</script>

<template>
  <div class="games-toolbar">
    <div class="game-tabs">
      <button
        v-for="game in games"
        :key="game.id"
        type="button"
        class="game-tab"
        :class="{ active: game.id === selectedGame?.id }"
        @click="emit('select-game', game.id)"
      >
        {{ game.id === selectedGame?.id ? pageTitle : game.gameName }}
      </button>
    </div>
    <div class="games-meta">
      <span>第 {{ (gamePage.page || 0) + 1 }} / {{ gamePage.totalPages || 1 }} 页</span>
      <span>{{ gamePage.total }} 个游戏</span>
    </div>
  </div>

  <div class="draw-strip">
    <div class="draw-strip-left">
      <span>第</span>
      <strong>{{ latestDrawIssueLabel }}</strong>
      <span>开奖结果:</span>
    </div>

    <div v-if="latestDrawNumbers.length === 1 && latestDrawNumbers[0] === '加载中'" class="draw-loading">
      开奖数据加载中...
    </div>

    <div v-else class="draw-balls">
      <span
        v-for="(num, index) in latestDrawNumbers"
        :key="`${latestDrawResultId}-${index}-${num}`"
        class="draw-ball"
        :class="{ active: index === latestDrawNumbers.length - 1 }"
      >
        {{ num }}
      </span>
    </div>

    <div class="draw-strip-right">{{ latestDrawSpecial }}番</div>
  </div>

  <p v-if="drawResultsError" class="console-message is-error">{{ drawResultsError }}</p>
  <p v-else-if="drawResultsLoading" class="console-message">开奖号码加载中...</p>
  <p v-if="winResultsError" class="console-message is-error">{{ winResultsError }}</p>
  <p v-else-if="winResultsLoading" class="console-message">中奖结果加载中...</p>

  <div class="issue-banner">
    <span class="issue-banner-playmode">
      玩法：
      <button
        v-for="mode in availablePlayModes"
        :key="mode.value"
        type="button"
        class="playmode-tab"
        :class="{ active: mode.value === selectedPlayMode }"
        @click="emit('change-play-mode', mode.value)"
      >
        {{ mode.label }}
      </button>
      <template v-if="!availablePlayModes.length">{{ currentPlayModeLabel }}</template>
    </span>

    <span v-if="showBallSelector" class="issue-banner-ball">
      球位：
      <button
        v-for="ball in ballOptions"
        :key="ball.value"
        type="button"
        class="playmode-tab"
        :class="{ active: ball.value === selectedBall }"
        @click="emit('select-ball', ball.value)"
      >
        {{ ball.label }}
      </button>
    </span>

    <span>期数：{{ selectedGame?.currentIssueNo || '--' }}</span>
    <span>{{ selectedGame?.sealed ? '封盘中' : '未封盘' }}</span>
    <span>{{ currentIssueStatusText }}</span>
    <span>开奖时间：{{ formatDateTime(selectedGame?.currentDrawTime) }}</span>
  </div>

  <p v-if="playSettingsError" class="console-message is-error">{{ playSettingsError }}</p>
  <p v-else-if="playSettingsLoading" class="console-message">玩法赔率加载中...</p>

  <div class="console-board-wrap">
    <section class="betting-board">
      <div class="board-header">
        <div>
          <strong>{{ pageTitle || selectedGame?.gameName || '未选择游戏' }}</strong>
        </div>
        <div>
          <strong>{{ selectedGame?.sealed ? '已封盘' : '可下注' }}</strong>
          <span>{{ formatDateTime(selectedGame?.serverTime) }}</span>
        </div>
      </div>

      <div class="board-shell">
        <section class="board-column board-left">
          <article
            class="corner-box board-clickable"
            :class="{ active: isPlaySelected('角1 2') }"
            @click="emit('select-bet-play', '角1 2')"
          >
            <div class="corner-title">{{ getCornerTokens('角1 2').label }}</div>
            <div class="corner-value-row">
              <span>{{ getCornerTokens('角1 2').left }}</span>
              <span>{{ getCornerTokens('角1 2').right }}</span>
            </div>
            <div class="corner-odds">{{ getPlayOdds('角1 2') }}</div>
          </article>

          <div class="side-panel">
            <article
              class="side-straight-box board-clickable"
              :class="{ active: isPlaySelected('2正') }"
              @click="emit('select-bet-play', '2正')"
            >
              <div class="side-straight-name">2正</div>
              <div class="side-straight-odds">{{ getPlayOdds('2正') }}</div>
            </article>

            <div class="side-play-list">
              <article
                v-for="playName in leftTriplePlays"
                :key="playName"
                class="board-play-box board-clickable"
                :class="{ active: isPlaySelected(playName) }"
                @click="emit('select-bet-play', playName)"
              >
                <div class="board-play-name">{{ playName }}</div>
                <div class="board-play-odds">{{ getPlayOdds(playName) }}</div>
              </article>
            </div>
          </div>

          <article
            class="corner-box board-clickable"
            :class="{ active: isPlaySelected('角2 3') }"
            @click="emit('select-bet-play', '角2 3')"
          >
            <div class="corner-title">{{ getCornerTokens('角2 3').label }}</div>
            <div class="corner-value-row">
              <span>{{ getCornerTokens('角2 3').left }}</span>
              <span>{{ getCornerTokens('角2 3').right }}</span>
            </div>
            <div class="corner-odds">{{ getPlayOdds('角2 3') }}</div>
          </article>
        </section>

        <section class="board-column board-center-column">
          <article
            class="straight-box board-clickable"
            :class="{ active: isPlaySelected('1正') }"
            @click="emit('select-bet-play', '1正')"
          >
            <div class="straight-name">1正</div>
            <div class="straight-odds">{{ getPlayOdds('1正') }}</div>
          </article>

          <div class="board-row triple-row">
            <article
              v-for="playName in topTriplePlays"
              :key="playName"
              class="board-play-box board-clickable"
              :class="{ active: isPlaySelected(playName) }"
              @click="emit('select-bet-play', playName)"
            >
              <div class="board-play-name">{{ playName }}</div>
              <div class="board-play-odds">{{ getPlayOdds(playName) }}</div>
            </article>
          </div>

          <div class="cross-board">
            <article
              class="cross-area board-clickable cross-area-top"
              :class="{ active: isPlaySelected('1番') }"
              @click="emit('select-bet-play', '1番')"
            >
              <div class="cross-area-content cross-area-content-top">
                <div class="board-play-name board-play-name-large">1番</div>
                <div class="board-play-odds">{{ getPlayOdds('1番') }}</div>
              </div>
            </article>

            <article
              class="cross-area board-clickable cross-area-left"
              :class="{ active: isPlaySelected('2番') }"
              @click="emit('select-bet-play', '2番')"
            >
              <div class="cross-area-content cross-area-content-left">
                <div class="board-play-name board-play-name-large">2番</div>
                <div class="board-play-odds">{{ getPlayOdds('2番') }}</div>
              </div>
            </article>

            <article
              class="cross-area board-clickable cross-area-right"
              :class="{ active: isPlaySelected('4番') }"
              @click="emit('select-bet-play', '4番')"
            >
              <div class="cross-area-content cross-area-content-right">
                <div class="board-play-name board-play-name-large">4番</div>
                <div class="board-play-odds">{{ getPlayOdds('4番') }}</div>
              </div>
            </article>

            <article
              class="cross-area board-clickable cross-area-bottom"
              :class="{ active: isPlaySelected('3番') }"
              @click="emit('select-bet-play', '3番')"
            >
              <div class="cross-area-content cross-area-content-bottom">
                <div class="board-play-name board-play-name-large">3番</div>
                <div class="board-play-odds">{{ getPlayOdds('3番') }}</div>
              </div>
            </article>

            <svg
              class="cross-lines"
              viewBox="0 0 100 100"
              preserveAspectRatio="none"
              aria-hidden="true"
            >
              <line x1="0" y1="0" x2="50" y2="50" />
              <line x1="100" y1="0" x2="50" y2="50" />
              <line x1="0" y1="100" x2="50" y2="50" />
              <line x1="100" y1="100" x2="50" y2="50" />
            </svg>
          </div>

          <div class="board-row triple-row">
            <article
              v-for="playName in bottomTriplePlays"
              :key="playName"
              class="board-play-box board-clickable"
              :class="{ active: isPlaySelected(playName) }"
              @click="emit('select-bet-play', playName)"
            >
              <div class="board-play-name">{{ playName }}</div>
              <div class="board-play-odds">{{ getPlayOdds(playName) }}</div>
            </article>
          </div>

          <article
            class="straight-box board-clickable"
            :class="{ active: isPlaySelected('3正') }"
            @click="emit('select-bet-play', '3正')"
          >
            <div class="straight-name">3正</div>
            <div class="straight-odds">{{ getPlayOdds('3正') }}</div>
          </article>
        </section>

        <section class="board-column board-right">
          <article
            class="corner-box board-clickable"
            :class="{ active: isPlaySelected('角1 4') }"
            @click="emit('select-bet-play', '角1 4')"
          >
            <div class="corner-title">{{ getCornerTokens('角1 4').label }}</div>
            <div class="corner-value-row">
              <span>{{ getCornerTokens('角1 4').left }}</span>
              <span>{{ getCornerTokens('角1 4').right }}</span>
            </div>
            <div class="corner-odds">{{ getPlayOdds('角1 4') }}</div>
          </article>

          <div class="side-panel side-panel-right">
            <div class="side-play-list">
              <article
                v-for="playName in rightTriplePlays"
                :key="playName"
                class="board-play-box board-clickable"
                :class="{ active: isPlaySelected(playName) }"
                @click="emit('select-bet-play', playName)"
              >
                <div class="board-play-name">{{ playName }}</div>
                <div class="board-play-odds">{{ getPlayOdds(playName) }}</div>
              </article>
            </div>

            <article
              class="side-straight-box board-clickable"
              :class="{ active: isPlaySelected('4正') }"
              @click="emit('select-bet-play', '4正')"
            >
              <div class="side-straight-name">4正</div>
              <div class="side-straight-odds">{{ getPlayOdds('4正') }}</div>
            </article>
          </div>

          <article
            class="corner-box board-clickable"
            :class="{ active: isPlaySelected('角3 4') }"
            @click="emit('select-bet-play', '角3 4')"
          >
            <div class="corner-title">{{ getCornerTokens('角3 4').label }}</div>
            <div class="corner-value-row">
              <span>{{ getCornerTokens('角3 4').left }}</span>
              <span>{{ getCornerTokens('角3 4').right }}</span>
            </div>
            <div class="corner-odds">{{ getPlayOdds('角3 4') }}</div>
          </article>
        </section>
      </div>
    </section>

    <aside class="trend-panel">
      <div class="trend-title">号码走势</div>
      <div class="trend-table">
        <template v-for="(row, rowIndex) in trendRows" :key="`row-${rowIndex}`">
          <div
            v-for="(cell, colIndex) in row"
            :key="`cell-${rowIndex}-${colIndex}`"
            class="trend-cell"
            :class="{ accent: cell % 2 === 0 }"
          >
            {{ cell }}
          </div>
        </template>
      </div>
    </aside>
  </div>

  <section class="lower-play-area">
    <div class="extra-play-grid">
      <div class="extra-play-left">
        <div class="extra-grid extra-grid-four">
          <article
            v-for="playName in zhongPlayNames"
            :key="playName"
            class="extra-play-card board-clickable"
            :class="{ active: isPlaySelected(playName) }"
            @click="emit('select-bet-play', playName)"
          >
            <div class="extra-play-name">{{ playName }}</div>
            <div class="extra-play-odds">{{ getPlayOdds(playName) }}</div>
          </article>
        </div>

        <div class="extra-grid extra-grid-four">
          <article
            v-for="playName in bigSmallOddEvenPlays"
            :key="playName"
            class="extra-play-card board-clickable"
            :class="{ active: isPlaySelected(playName) }"
            @click="emit('select-bet-play', playName)"
          >
            <div class="extra-play-name">{{ playName }}</div>
            <div class="extra-play-odds">{{ getPlayOdds(playName) }}</div>
          </article>
        </div>

        <div class="extra-grid extra-grid-four">
          <article
            v-for="playName in tongPlayNames"
            :key="playName"
            class="extra-play-card board-clickable"
            :class="{ active: isPlaySelected(playName) }"
            @click="emit('select-bet-play', playName)"
          >
            <div class="extra-play-name">{{ playName }}</div>
            <div class="extra-play-odds">{{ getPlayOdds(playName) }}</div>
          </article>
        </div>
      </div>

      <div class="extra-play-right">
        <div class="extra-grid extra-grid-two">
          <article
            v-for="playName in jiaPlayNames"
            :key="playName"
            class="extra-play-card board-clickable"
            :class="{ active: isPlaySelected(playName) }"
            @click="emit('select-bet-play', playName)"
          >
            <div class="extra-play-name">{{ playName }}</div>
            <div class="extra-play-odds">{{ getPlayOdds(playName) }}</div>
          </article>
        </div>
      </div>
    </div>

    <div v-if="showTemaSection" class="tema-grid-wrap">
      <div class="tema-grid">
        <article
          v-for="playName in temaNumberPlays"
          :key="playName"
          class="tema-number-card board-clickable"
          :class="{ active: isPlaySelected(playName) }"
          @click="emit('select-bet-play', playName)"
        >
          <div class="tema-ball" :class="{ red: playName === '19' || playName === '20' }">
            {{ playName }}
          </div>
          <div class="tema-odds">{{ getPlayOdds(playName) }}</div>
        </article>

        <article
          v-for="playName in visibleTemaSidePlays"
          :key="playName"
          class="tema-side-card board-clickable"
          :class="{ active: isPlaySelected(playName) }"
          @click="emit('select-bet-play', playName)"
        >
          <div class="extra-play-name">{{ playName }}</div>
          <div class="extra-play-odds">{{ getPlayOdds(playName) }}</div>
        </article>
      </div>
    </div>
  </section>

</template>
