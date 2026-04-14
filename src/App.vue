<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref } from 'vue'
import Game1Page from './components/Game1Page.vue'
import Game2Page from './components/Game2Page.vue'
import Game3Page from './components/Game3Page.vue'
import Game4Page from './components/Game4Page.vue'
import Game5Page from './components/Game5Page.vue'
import Game6Page from './components/Game6Page.vue'
import Game7Page from './components/Game7Page.vue'

const STORAGE_KEY = 'member-login-session'
const AGREEMENT_KEY = 'member-agreement-accepted'

const agreementItems = [
  '01.为避免出现争议，请您务必在下注之后查看“下注状况”。',
  '02.任何投诉必须在开奖之前，后台将不接受任何开奖之后的投诉。',
  '03.公布赔率时出现的任何打字错误或非故意人为失误，所有（相关）注单一律不算。',
  '04.会员版面公布的为浮动赔率，与下注实际赔率会有差异，属正常浮动。下注成功后请确认注单赔率及金额！',
  '05.开奖后接受的投注，一律视为无效。',
  '06.若本后台发现客户以不正当的手法投注或投注注单不正常，后台将有权“取消”相应之注单，客户不得有任何异议。',
  '07.如因软件或线路问题导致交易内容或其他与账号设定不符合的情形，请在开奖前立即与本后台联络反映问题，否则本后台将以资料库中的数据为准。',
  '08.如因服务供应商或线路问题导致赛果与官方赛果不符合的情形。本后台将竭力调查以确定赛果，除非后台所发布的赛果明显错误，否则本后台的决定将是最终决定。受影响之注单将重新结算，后台将以重新结算的数据为准，客户不得有任何异议。',
  '09.倘若发生遭黑客入侵破坏行为或不可抗拒之灾害致网站故障或资料损坏、数据丢失等情况，后台将以资料库数据为依据。',
  '10.各级管理人员及客户必须对本系统各项功能进行了解及熟悉，任何违反正常使用的操作，后台概不负责。',
  '11.请认真了解游戏规则。',
  '12.如果会员信用额度超额或者为负数引起的争议，一律以注单报表为准。',
  '13.客户有责任确保自己的账户及密码保密，如果因客户的账户、密码简单，或因泄露导致被盗用，造成的损失由客户本人承担；同时应立即通知本公司，并更改其个人详细资料。',
  '14.会员账号余额只做参考不做交收依据,一律以报表为准。',
]

const noticeText =
  '公告：会员控制台已接入当前用户信息与游戏分页列表接口，赔率格与走势表暂以展示布局为主。'

const quickActions = ['游戏投注', '下注明细']
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
const temaSidePlays = ['特大', '特小', '特单', '特双']

const readStoredSession = () => {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    return raw ? JSON.parse(raw) : null
  } catch {
    return null
  }
}

const form = reactive({
  username: '',
  password: '',
})

const normalizeLoginSession = (payload) => {
  const source = payload?.data && typeof payload.data === 'object' ? payload.data : payload
  if (!source || typeof source !== 'object') {
    return null
  }

  return {
    userId: source.userId ?? null,
    username: source.username ?? '',
    nickname: source.nickname ?? '',
    userType: source.userType ?? 'member',
    adminSide: source.adminSide ?? false,
    sidePlayModeMask: source.sidePlayModeMask ?? null,
    token: source.token ?? '',
    expiresInSeconds: source.expiresInSeconds ?? null,
    expiresAtEpochSecond: source.expiresAtEpochSecond ?? null,
  }
}

const view = ref('login')
const loading = ref(false)
const errorMessage = ref('')
const successMessage = ref('')
const loginResult = ref(readStoredSession())

const dashboardLoading = ref(false)
const dashboardError = ref('')
const currentUser = ref(null)
const games = ref([])
const drawResults = ref([])
const drawResultsLoading = ref(false)
const drawResultsError = ref('')
const winResults = ref([])
const winResultsLoading = ref(false)
const winResultsError = ref('')
const backgroundRefreshing = ref(false)
const playSettings = ref([])
const playSettingsLoading = ref(false)
const playSettingsError = ref('')
const selectedPlayMode = ref('')
const selectedBetPlay = ref('')
const betSlipItems = ref([])
const betSubmitError = ref('')
const betSubmitMessage = ref('')
const betSubmitting = ref(false)
const activeQuickAction = ref('游戏投注')
const bets = ref([])
const betsLoading = ref(false)
const betsError = ref('')
const betsPage = ref({
  total: 0,
  page: 0,
  size: 6,
  totalPages: 0,
})
const gamePage = ref({
  total: 0,
  page: 0,
  size: 12,
  totalPages: 0,
})
const selectedGameId = ref(null)
const selectedBall = ref(1)
const countdownSeconds = ref(0)

let countdownTimer = null
let dashboardPollTimer = null

const getAuthToken = () => loginResult.value?.token || ''

const compareIssueNo = (left, right) => {
  const leftText = String(left || '').trim()
  const rightText = String(right || '').trim()

  if (!leftText && !rightText) {
    return 0
  }

  const leftDigits = leftText.replace(/\D/g, '')
  const rightDigits = rightText.replace(/\D/g, '')

  if (leftDigits && rightDigits) {
    if (leftDigits.length !== rightDigits.length) {
      return leftDigits.length - rightDigits.length
    }
    return leftDigits.localeCompare(rightDigits)
  }

  return leftText.localeCompare(rightText)
}

const formatDateTime = (value) => {
  if (!value) {
    return '--'
  }

  const text = String(value).trim()
  if (!text) {
    return '--'
  }

  return text.replace('T', ' ').replace(/\.\d+$/, '')
}

const displayName = computed(() => {
  const name =
    currentUser.value?.username ||
    loginResult.value?.nickname ||
    loginResult.value?.username
  return name || '会员'
})

const sidePlayModeMaskValue = computed(() => {
  return Number(currentUser.value?.sidePlayModeMask ?? loginResult.value?.sidePlayModeMask ?? 0)
})

const availablePlayModes = computed(() => {
  const mask = sidePlayModeMaskValue.value
  const modes = []

  if (mask & 1) {
    modes.push({ value: 'single', label: '单边' })
  }
  if (mask & 2) {
    modes.push({ value: 'double', label: '双边' })
  }

  return modes
})

const currentPlayModeLabel = computed(() => {
  return availablePlayModes.value.find((item) => item.value === selectedPlayMode.value)?.label || '--'
})

const displayedBets = computed(() => bets.value)

const activeBetSlipItem = computed(() => {
  return betSlipItems.value.find((item) => item.playName === selectedBetPlay.value) || null
})

const currentMemberId = computed(() => {
  return (
    activeBetSlipItem.value?.memberId ||
    playSettings.value[0]?.memberId ||
    currentUser.value?.memberId ||
    currentUser.value?.id ||
    loginResult.value?.userId ||
    null
  )
})

const playSettingsMap = computed(() => {
  return playSettings.value.reduce((accumulator, item) => {
    if (item?.playName) {
      accumulator[item.playName] = item.odds
    }
    return accumulator
  }, {})
})

const normalizedPlaySettingsMap = computed(() => {
  return playSettings.value.reduce((accumulator, item) => {
    const key = normalizePlayName(item?.playName)
    if (key) {
      accumulator[key] = item.odds
    }
    return accumulator
  }, {})
})

const playSettingAliasGroups = {
  大小单双: ['大小单双', '大小,單雙', '大小单双', '大小单双'],
  特码大小单双: ['特码大小单双', '特码大单、小双', '特大特小特单双', '特大小特单双'],
  特码大单小双: ['特码大单、小双', '特码大小单双', '特大特小特单双', '特大小特单双'],
  特码: ['特码', '特碼'],
  番: ['番', '翻'],
}

const selectedGame = computed(() => {
  if (!games.value.length) {
    return null
  }
  return (
    games.value.find((item) => item.id === selectedGameId.value) || games.value[0]
  )
})

const countdownText = computed(() => {
  const total = Math.max(0, countdownSeconds.value || 0)
  const hours = String(Math.floor(total / 3600)).padStart(2, '0')
  const minutes = String(Math.floor((total % 3600) / 60)).padStart(2, '0')
  const seconds = String(total % 60).padStart(2, '0')
  return `${hours}:${minutes}:${seconds}`
})

const latestDrawResult = computed(() => {
  const finishedResults = drawResults.value.filter((item) => {
    return item?.drawCode !== null && item?.drawCode !== undefined && String(item.drawCode).trim()
  })

  if (!finishedResults.length) {
    return null
  }

  const currentIssueNo = selectedGame.value?.currentIssueNo
  if (!currentIssueNo) {
    return finishedResults[0]
  }

  const previousResult = finishedResults.find((item) => {
    return compareIssueNo(item.issueNo, currentIssueNo) < 0
  })

  return previousResult || finishedResults[0]
})

const currentWinResult = computed(() => {
  const currentIssueNo = selectedGame.value?.currentIssueNo
  if (!currentIssueNo) {
    return winResults.value[0] || null
  }

  return (
    winResults.value.find((item) => String(item.issueNo || '') === String(currentIssueNo)) ||
    winResults.value[0] ||
    null
  )
})

const displayWinResult = computed(() => {
  if (currentWinResult.value?.drawCode) {
    return currentWinResult.value
  }

  return latestDrawResult.value
})

const latestDrawNumbers = computed(() => {
  const drawCode = displayWinResult.value?.drawCode
  if (!drawCode) {
    return []
  }

  const parts = String(drawCode)
    .trim()
    .split(/[^0-9A-Za-z]+/)
    .filter(Boolean)

  if (parts.length > 1) {
    return parts
  }

  return String(drawCode)
    .trim()
    .split('')
    .filter(Boolean)
})

const latestDrawSpecial = computed(() => {
  const fanText = String(displayWinResult.value?.fanResult || '').trim()
  const matched = fanText.match(/([1-4])/)
  return matched ? matched[1] : '--'
})

const latestDrawIssueLabel = computed(() => {
  return displayWinResult.value?.issueNo || '--'
})

const currentIssueStatusText = computed(() => {
  if (!selectedGame.value) {
    return '资料加载中...'
  }

  if (selectedGame.value.sealed) {
    return '已封盘'
  }

  return `距离开奖${countdownText.value}`
})

const formatOdds = (value) => {
  if (value === null || value === undefined || value === '') {
    return '--'
  }

  const numericValue = Number(value)
  if (Number.isNaN(numericValue)) {
    return String(value)
  }

  return numericValue.toFixed(4)
}

const formatMoney = (value) => {
  if (value === null || value === undefined || value === '') {
    return '0.00'
  }

  const numericValue = Number(value)
  if (Number.isNaN(numericValue)) {
    return '0.00'
  }

  return numericValue.toFixed(2)
}

const normalizePlayName = (value) => {
  return String(value || '')
    .trim()
    .replace(/[，、]/g, ',')
    .replace(/\s+/g, '')
    .replace(/單/g, '单')
    .replace(/雙/g, '双')
    .replace(/翻/g, '番')
}

const resolvePlayOddsKey = (playName) => {
  const text = normalizePlayName(playName)

  if (!text) {
    return ''
  }

  if (text.includes('角')) {
    return '角'
  }

  if (text.includes('正')) {
    return '正'
  }

  if (text.includes('念')) {
    return '念'
  }

  if (text.includes('中')) {
    return '中'
  }

  if (text.includes('通')) {
    return '通'
  }

  if (text.includes('加')) {
    return '加'
  }

  if (/^\d+$/.test(text)) {
    return '特码'
  }

  if (text.startsWith('特')) {
    return '特码大单、小双'
  }

  if (text === '单' || text === '双' || text === '大' || text === '小') {
    return '大小,單雙'
  }

  if (text === '大单' || text === '大双' || text === '小单' || text === '小双') {
    return '大小单双'
  }

  if (text.includes('番')) {
    return '番'
  }

  return text
}

const getPlayOdds = (playName) => {
  const key = resolvePlayOddsKey(playName)
  const normalizedKey = normalizePlayName(key)
  const aliases = playSettingAliasGroups[normalizedKey] || [key]
  for (const alias of aliases) {
    if (playSettingsMap.value[alias] !== undefined) {
      return formatOdds(playSettingsMap.value[alias])
    }

    const normalizedAlias = normalizePlayName(alias)
    if (normalizedPlaySettingsMap.value[normalizedAlias] !== undefined) {
      return formatOdds(normalizedPlaySettingsMap.value[normalizedAlias])
    }
  }

  return formatOdds(playSettingsMap.value[key] ?? normalizedPlaySettingsMap.value[normalizedKey])
}

const findPlaySettingByPlayName = (playName) => {
  const key = resolvePlayOddsKey(playName)
  const normalizedKey = normalizePlayName(key)
  const aliases = [key, ...(playSettingAliasGroups[normalizedKey] || [key])]

  for (const alias of aliases) {
    const normalizedAlias = normalizePlayName(alias)
    const matched = playSettings.value.find(
      (item) => normalizePlayName(item?.playName) === normalizedAlias,
    )
    if (matched) {
      return matched
    }
  }

  return null
}

const selectedBetPlaySetting = computed(() => {
  if (!selectedBetPlay.value) {
    return null
  }

  return findPlaySettingByPlayName(selectedBetPlay.value)
})

const selectedBetOdds = computed(() => {
  return activeBetSlipItem.value?.odds || '--'
})

const getBetOddsDisplay = (bet) => {
  if (bet?.odds !== null && bet?.odds !== undefined && bet?.odds !== '') {
    return formatOdds(bet.odds)
  }

  return getPlayOdds(bet?.playName || bet?.selectionValue || '')
}

const buildSelectionValue = (playName) => {
  const text = normalizePlayName(playName)
  if (!text) {
    return ''
  }

  if (/^\d+正$/.test(text) || /^\d+番$/.test(text)) {
    return text.replace(/\D/g, '')
  }

  if (text.includes('角')) {
    const digits = text.replace(/\D/g, '').slice(0, 2)
    return digits ? `${digits}角` : ''
  }

  if (/^\d念\d$/.test(text) || /^\d加\d{2}$/.test(text) || /^\d{2}[一二三四]通$/.test(text) || /^\d{3}中$/.test(text)) {
    return playName
  }

  if (/^\d+$/.test(text)) {
    return text
  }

  if (
    text === '特大单' ||
    text === '特小单' ||
    text === '特大双' ||
    text === '特小双'
  ) {
    return text.replace(/^特/, '')
  }

  if (text === '特大' || text === '特小' || text === '特单' || text === '特双') {
    return text.replace(/^特/, '')
  }

  return playName
}

const normalizedSelectionValue = computed(() => {
  return buildSelectionValue(selectedBetPlay.value)
})

const betCount = computed(() => betSlipItems.value.length)
const betTotalAmount = computed(() => {
  const total = betSlipItems.value.reduce((sum, item) => {
    const numericValue = Number(item.betAmount)
    return sum + (Number.isFinite(numericValue) && numericValue > 0 ? numericValue : 0)
  }, 0)

  return formatMoney(total)
})
const betSubmitButtonText = computed(() => `${currentPlayModeLabel.value || '确认'}下注`)

const getCornerTokens = (playName) => {
  const text = String(playName || '').trim()
  const match = text.match(/(\d)\s*角\s*(\d)|角\s*(\d)\s*(\d)/)

  if (match) {
    return {
      label: '角',
      left: match[1] || match[3] || '',
      right: match[2] || match[4] || '',
    }
  }

  return {
    label: '角',
    left: text.replace(/\D/g, '').charAt(0) || '',
    right: text.replace(/\D/g, '').charAt(1) || '',
  }
}

const formatBallIndexLabel = (ballIndex) => {
  if (gamesWithoutBallSelector.includes(Number(selectedGame.value?.id || 0))) {
    return '-'
  }
  const numericValue = Number(ballIndex)
  return Number.isFinite(numericValue) && numericValue > 0 ? `${numericValue}球` : '--'
}

const parseDrawCodeNumbers = (drawCode) => {
  if (!drawCode) {
    return []
  }

  return String(drawCode)
    .trim()
    .split(/[^0-9A-Za-z]+/)
    .filter(Boolean)
}

const extractResultTail = (resultDetail) => {
  const text = String(resultDetail || '').trim()
  if (!text) {
    return ''
  }

  const normalizedText = normalizePlayName(text)
  const matched =
    normalizedText.match(/([1-4]番)/) ||
    normalizedText.match(/(大单|小单|大双|小双|大|小|单|双)/) ||
    normalizedText.match(/(\d+特)/)

  return matched ? matched[1] : ''
}

const getBetDetailDisplay = (bet) => {
  const gameId = Number(bet?.gameId || selectedGame.value?.id || 0)
  if (bet?.betDetail) {
    const detailText = String(bet.betDetail).trim()
    if (gamesWithoutBallSelector.includes(gameId)) {
      return detailText.replace(/[一二三四五六七八九十\d]+球\s*/g, '').trim() || detailText
    }
    return detailText
  }

  const ballLabel = formatBallIndexLabel(bet?.ballIndex)
  const playLabel = bet?.selectionValue || bet?.playName || '--'
  return gamesWithoutBallSelector.includes(gameId) ? playLabel : `${ballLabel}${playLabel}`
}

const escapeRegExp = (value) => {
  return String(value || '').replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
}

const getBetDetailWithoutIssue = (bet) => {
  const detailText = String(getBetDetailDisplay(bet) || '').trim()
  if (!detailText) {
    return '--'
  }

  const withoutLeadingIssue = detailText
    .replace(/^\d+\s*[期號号]?\s*/, '')
    .replace(/^\d+\s+/, '')
    .trim()

  const playName = String(bet?.playName || '').trim()
  if (!playName) {
    return withoutLeadingIssue || detailText
  }

  const ballPrefixMatch = withoutLeadingIssue.match(/^([一二三四五六七八九十\d]+球)\s*(.+)$/)
  if (ballPrefixMatch) {
    const [, ballLabel, restText] = ballPrefixMatch
    const withoutPlayName = restText.replace(new RegExp(`^${escapeRegExp(playName)}\\s*`), '').trim()
    return withoutPlayName ? `${ballLabel} ${withoutPlayName}` : withoutLeadingIssue
  }

  const withoutPlayName = withoutLeadingIssue
    .replace(new RegExp(`^${escapeRegExp(playName)}\\s*`), '')
    .trim()

  return withoutPlayName || withoutLeadingIssue || detailText
}

const getBetResultDisplay = (bet) => {
  const numbers = parseDrawCodeNumbers(bet?.drawCode)
  const tail = extractResultTail(bet?.resultDetail)

  if (!numbers.length && !tail) {
    return null
  }

  return {
    title: getBetDetailWithoutIssue(bet),
    numbers,
    highlightIndex: Math.max(0, Number(bet?.ballIndex || 1) - 1),
    tail,
  }
}

const trendRows = computed(() => {
  const rowOrder = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0]
  const groupedByTail = rowOrder.reduce((accumulator, tail) => {
    accumulator[tail] = []
    return accumulator
  }, {})

  const orderedResults = [...winResults.value].sort((left, right) => compareIssueNo(left?.issueNo, right?.issueNo))

  orderedResults.forEach((item) => {
    const issueText = String(item?.issueNo || '').trim()
    const issueTailText = issueText.slice(-1)
    const issueTail = Number(issueTailText)
    const fanMatch = String(item?.fanResult || '').match(/([1-4])/)
    const fanValue = fanMatch ? Number(fanMatch[1]) : null

    if (!Number.isNaN(issueTail) && fanValue !== null && groupedByTail[issueTail]) {
      groupedByTail[issueTail].push(fanValue)
    }
  })

  const columnCount = Math.max(12, ...rowOrder.map((tail) => groupedByTail[tail].length))
  return rowOrder.map((tail) => {
    const rowValues = groupedByTail[tail]
    return Array.from({ length: columnCount }, (_, index) => rowValues[index] ?? '')
  })
})

const gameBallConfig = {
  1: [1],
  2: [8],
  3: [2, 3, 4, 5, 6, 7],
  7: [1, 2, 3, 4, 5, 6, 7, 8],
}

const ballLabelMap = {
  1: '第一球',
  2: '第二球',
  3: '第三球',
  4: '第四球',
  5: '第五球',
  6: '第六球',
  7: '第七球',
  8: '第八球',
}

const gamesWithoutBallSelector = [4, 5, 6]

const ballOptions = computed(() => {
  const gameId = Number(selectedGame.value?.id || 0)
  const supportedBalls = gameBallConfig[gameId] || [1]
  return supportedBalls.map((ball) => ({
    value: ball,
    label: ballLabelMap[ball] || `${ball}球`,
  }))
})

const currentGamePageTitle = computed(() => {
  const gameName = String(selectedGame.value?.gameName || '').trim()
  if (!gameName) {
    return '当前游戏1球'
  }

  if (gamesWithoutBallSelector.includes(Number(selectedGame.value?.id || 0))) {
    return gameName
  }

  const currentBallLabel = ballOptions.value.find((item) => item.value === selectedBall.value)?.label || ''
  return currentBallLabel ? `${gameName}-${currentBallLabel}` : gameName
})

const currentGamePageComponent = computed(() => {
  const gameId = Number(selectedGame.value?.id || 0)
  const gameComponentMap = {
    1: Game1Page,
    2: Game2Page,
    3: Game3Page,
    4: Game4Page,
    5: Game5Page,
    6: Game6Page,
    7: Game7Page,
  }

  return gameComponentMap[gameId] || Game1Page
})

const apiFetch = async (url, options = {}) => {
  const headers = new Headers(options.headers || {})
  if (!headers.has('Content-Type') && options.body) {
    headers.set('Content-Type', 'application/json')
  }

  const token = getAuthToken()
  if (token) {
    headers.set('Authorization', `Bearer ${token}`)
  }

  const response = await fetch(url, {
    ...options,
    headers,
  })

  const payload = await response.json().catch(() => ({}))
  if (!response.ok || payload.success === false) {
    throw new Error(payload.message || '接口请求失败')
  }

  return payload
}

const startCountdown = (seconds) => {
  window.clearInterval(countdownTimer)
  countdownSeconds.value = Number(seconds) || 0

  if (countdownSeconds.value <= 0) {
    return
  }

  countdownTimer = window.setInterval(() => {
    if (countdownSeconds.value <= 0) {
      window.clearInterval(countdownTimer)
      return
    }
    countdownSeconds.value -= 1
  }, 1000)
}

const loadDashboard = async (silent = false) => {
  if (!getAuthToken()) {
    dashboardError.value = '缺少登录令牌，请重新登录'
    return
  }

  if (!silent) {
    dashboardLoading.value = true
    dashboardError.value = ''
  }

  backgroundRefreshing.value = silent

  try {
    const [currentPayload, gamesPayload] = await Promise.all([
      apiFetch('/api/member/current'),
      apiFetch('/api/member/games?page=0&size=12'),
    ])

    currentUser.value = currentPayload.data || null
    if (
      !availablePlayModes.value.some((item) => item.value === selectedPlayMode.value)
    ) {
      selectedPlayMode.value = availablePlayModes.value[0]?.value || ''
    }

    const pageData = gamesPayload.data || gamesPayload
    const gameList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    games.value = Array.isArray(gameList) ? gameList : []
    gamePage.value = {
      total: pageData.total || 0,
      page: pageData.page || 0,
      size: pageData.size || 12,
      totalPages: pageData.totalPages || 0,
    }

    if (games.value.length) {
      const previousGameId = selectedGameId.value
      selectedGameId.value =
        games.value.find((item) => item.id === selectedGameId.value)?.id ||
        games.value[0].id
      {
        const supportedBalls = gameBallConfig[Number(selectedGameId.value)] || [1]
        if (!supportedBalls.includes(selectedBall.value)) {
          selectedBall.value = supportedBalls[0]
        }
      }
      startCountdown(selectedGame.value?.sealCountdownSeconds)
      await loadDrawResults(selectedGameId.value, silent)
      await loadWinResults(selectedGameId.value, silent)
      await loadBets(selectedGameId.value, silent ? betsPage.value.page : 0, silent)
      if (!silent || previousGameId !== selectedGameId.value || !playSettings.value.length) {
        await loadPlaySettings(selectedGameId.value, silent)
      }
    } else {
      selectedGameId.value = null
      startCountdown(0)
      drawResults.value = []
      drawResultsError.value = ''
      winResults.value = []
      winResultsError.value = ''
      bets.value = []
      betsError.value = ''
      playSettings.value = []
      playSettingsError.value = ''
    }
  } catch (error) {
    dashboardError.value = error instanceof Error ? error.message : '控制台加载失败'
  } finally {
    dashboardLoading.value = false
    backgroundRefreshing.value = false
  }
}

const loadPlaySettings = async (gameId, silent = false) => {
  if (!gameId) {
    playSettings.value = []
    playSettingsError.value = ''
    return
  }

  if (!silent) {
    playSettingsLoading.value = true
    playSettingsError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/member/game-play-settings?gameId=${gameId}&page=0&size=50`)
    const pageData = payload.data || payload
    const settingsList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    playSettings.value = Array.isArray(settingsList) ? settingsList : []
  } catch (error) {
    if (!silent) {
      playSettings.value = []
      playSettingsError.value =
        error instanceof Error ? error.message : '玩法赔率加载失败'
    }
  } finally {
    if (!silent) {
      playSettingsLoading.value = false
    }
  }
}

const loadBets = async (gameId, page = 0, silent = false) => {
  const query = new URLSearchParams({
    page: String(page),
    size: String(betsPage.value.size),
  })

  if (gameId) {
    query.set('gameId', String(gameId))
  }

  if (!silent) {
    betsLoading.value = true
    betsError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/bets?${query.toString()}`)
    const pageData = payload.data || payload
    const betList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    bets.value = Array.isArray(betList) ? betList : []
    betsPage.value = {
      total: pageData.total || 0,
      page: pageData.page || 0,
      size: pageData.size || betsPage.value.size,
      totalPages: pageData.totalPages || 0,
    }
  } catch (error) {
    if (!silent) {
      bets.value = []
      betsError.value = error instanceof Error ? error.message : '下注明细加载失败'
    }
  } finally {
    if (!silent) {
      betsLoading.value = false
    }
  }
}

const loadWinResults = async (gameId, silent = false) => {
  if (!gameId) {
    winResults.value = []
    winResultsError.value = ''
    return
  }

  const query = new URLSearchParams({
    gameId: String(gameId),
    page: '0',
    size: '50',
  })

  if (!silent) {
    winResultsLoading.value = true
    winResultsError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/member/game-win-results?${query.toString()}`)
    const pageData = payload.data || payload
    const resultList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    winResults.value = Array.isArray(resultList) ? resultList : []
  } catch (error) {
    if (!silent) {
      winResults.value = []
      winResultsError.value =
        error instanceof Error ? error.message : '中奖结果加载失败'
    }
  } finally {
    if (!silent) {
      winResultsLoading.value = false
    }
  }
}

const loadDrawResults = async (gameId, silent = false) => {
  if (!gameId) {
    drawResults.value = []
    drawResultsError.value = ''
    return
  }

  if (!silent) {
    drawResultsLoading.value = true
    drawResultsError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/member/game-draw-results?gameId=${gameId}&page=0&size=10`)
    const pageData = payload.data || payload
    const resultList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    drawResults.value = Array.isArray(resultList)
      ? [...resultList].sort((a, b) => {
          const hasCodeA = a?.drawCode !== null && a?.drawCode !== undefined && String(a.drawCode).trim()
          const hasCodeB = b?.drawCode !== null && b?.drawCode !== undefined && String(b.drawCode).trim()
          if (hasCodeA !== hasCodeB) {
            return hasCodeB ? 1 : -1
          }

          const timeA = new Date(a.drawTime || 0).getTime()
          const timeB = new Date(b.drawTime || 0).getTime()
          if (timeA !== timeB) {
            return timeB - timeA
          }

          return compareIssueNo(b.issueNo, a.issueNo)
        })
      : []
  } catch (error) {
    if (!silent) {
      drawResults.value = []
      drawResultsError.value =
        error instanceof Error ? error.message : '开奖历史加载失败'
    }
  } finally {
    if (!silent) {
      drawResultsLoading.value = false
    }
  }
}

const stopAutoRefresh = () => {
  window.clearInterval(dashboardPollTimer)
  dashboardPollTimer = null
}

const startAutoRefresh = () => {
  stopAutoRefresh()

  dashboardPollTimer = window.setInterval(async () => {
    if (view.value !== 'dashboard' || !selectedGameId.value || dashboardLoading.value) {
      return
    }

    await loadDashboard(true)
  }, 5000)
}

const handleSubmit = async () => {
  errorMessage.value = ''
  successMessage.value = ''

  if (!form.username.trim() || !form.password.trim()) {
    errorMessage.value = '请输入用户名和密码'
    return
  }

  loading.value = true

  try {
    const data = await apiFetch('/api/member/login', {
      method: 'POST',
      body: JSON.stringify({
        username: form.username.trim(),
        password: form.password,
      }),
      headers: {
        'Content-Type': 'application/json',
      },
    })

    const session = normalizeLoginSession(data)
    if (!session?.token) {
      throw new Error(data.message || '登录成功但未返回 token')
    }

    loginResult.value = session
    successMessage.value = data.message || '登录成功'
    view.value = 'agreement'
    localStorage.removeItem(AGREEMENT_KEY)

    localStorage.setItem(STORAGE_KEY, JSON.stringify(session))
  } catch (error) {
    errorMessage.value = error instanceof Error ? error.message : '登录失败，请稍后重试'
  } finally {
    loading.value = false
  }
}

const agreeProtocol = async () => {
  localStorage.setItem(AGREEMENT_KEY, '1')
  view.value = 'dashboard'
  activeQuickAction.value = '游戏投注'
  await loadDashboard()
  startAutoRefresh()
}

const rejectProtocol = () => {
  stopAutoRefresh()
  localStorage.removeItem(AGREEMENT_KEY)
  successMessage.value = ''
  view.value = 'login'
}

const backToAgreement = () => {
  stopAutoRefresh()
  view.value = 'agreement'
}

const changePlayMode = (mode) => {
  selectedPlayMode.value = mode
}

const selectQuickAction = async (action) => {
  activeQuickAction.value = action
  if (action === '下注明细') {
    await loadBets(selectedGameId.value, 0, false)
  }
}

const selectBetPlay = (playName) => {
  selectedBetPlay.value = playName
  betSubmitError.value = ''
  betSubmitMessage.value = ''

  const existingItem = betSlipItems.value.find((item) => item.playName === playName)
  if (existingItem) {
    return
  }

  const playSetting = findPlaySettingByPlayName(playName)
  betSlipItems.value.push({
    playName,
    odds: getPlayOdds(playName),
    betAmount: '',
    gamePlayId: playSetting?.gamePlayId || playSetting?.id || null,
    memberId: playSetting?.memberId || playSettings.value[0]?.memberId || null,
  })
}

const selectGame = async (gameId) => {
  selectedGameId.value = gameId
  const nextSupportedBalls = gameBallConfig[Number(gameId)] || [1]
  if (!gamesWithoutBallSelector.includes(Number(gameId))) {
    selectedBall.value = nextSupportedBalls.includes(selectedBall.value)
      ? selectedBall.value
      : nextSupportedBalls[0]
  }
  startCountdown(selectedGame.value?.sealCountdownSeconds)
  selectedBetPlay.value = ''
  betSlipItems.value = []
  await Promise.all([
    loadDrawResults(gameId, false),
    loadWinResults(gameId, false),
    loadBets(gameId, 0, false),
    loadPlaySettings(gameId, false),
  ])
}

const changeBetsPage = async (nextPage) => {
  if (nextPage < 0) {
    return
  }
  if (betsPage.value.totalPages > 0 && nextPage >= betsPage.value.totalPages) {
    return
  }

  await loadBets(selectedGameId.value, nextPage, false)
}

const refreshDashboard = async () => {
  await loadDashboard()
}

const selectBall = (ball) => {
  const nextBall = Number(ball)
  if (!ballOptions.value.some((item) => item.value === nextBall)) {
    return
  }

  selectedBall.value = nextBall
  betSubmitError.value = ''
  betSubmitMessage.value = ''
}

const updateBetSlipAmount = (playName, value) => {
  const targetItem = betSlipItems.value.find((item) => item.playName === playName)
  if (!targetItem) {
    return
  }

  targetItem.betAmount = value
  selectedBetPlay.value = playName
  betSubmitError.value = ''
  betSubmitMessage.value = ''
}

const removeBetSlipItem = (playName) => {
  betSlipItems.value = betSlipItems.value.filter((item) => item.playName !== playName)
  if (selectedBetPlay.value === playName) {
    selectedBetPlay.value = betSlipItems.value[betSlipItems.value.length - 1]?.playName || ''
  }
  betSubmitError.value = ''
  betSubmitMessage.value = ''
}

const clearBetSlip = () => {
  selectedBetPlay.value = ''
  betSlipItems.value = []
  betSubmitError.value = ''
  betSubmitMessage.value = ''
}

const submitBet = async () => {
  betSubmitError.value = ''
  betSubmitMessage.value = ''

  if (!currentMemberId.value) {
    betSubmitError.value = '缺少会员ID，无法下注'
    return
  }

  if (!selectedGame.value?.id) {
    betSubmitError.value = '缺少游戏信息，无法下注'
    return
  }

  if (!betSlipItems.value.length) {
    betSubmitError.value = '请先选择玩法'
    return
  }

  betSubmitting.value = true

  try {
    for (const item of betSlipItems.value) {
      const gamePlayId = item.gamePlayId
      const selectionValue = buildSelectionValue(item.playName)
      const betAmount = Number(item.betAmount)

      if (!gamePlayId) {
        throw new Error(`玩法 ${item.playName} 缺少 gamePlayId，无法提交下注`)
      }

      if (!selectionValue) {
        throw new Error(`玩法 ${item.playName} 缺少下注值，无法提交`)
      }

      if (!Number.isFinite(betAmount) || betAmount <= 0) {
        throw new Error(`玩法 ${item.playName} 请输入正确的下注金额`)
      }

      await apiFetch('/api/bets', {
        method: 'POST',
        body: JSON.stringify({
          memberId: item.memberId || currentMemberId.value,
          gameId: selectedGame.value.id,
          gamePlayId,
          ...(gamesWithoutBallSelector.includes(Number(selectedGame.value.id))
            ? {}
            : { ballIndex: selectedBall.value }),
          selectionValue,
          betAmount: Number(formatMoney(betAmount)),
        }),
      })
    }

    betSubmitMessage.value = '下注成功'
    betSlipItems.value = []
    selectedBetPlay.value = ''
    await Promise.all([
      loadBets(selectedGameId.value, 0, false),
      apiFetch('/api/member/current').then((result) => {
        currentUser.value = result.data || currentUser.value
      }),
    ])
  } catch (error) {
    betSubmitError.value = error instanceof Error ? error.message : '下注失败'
  } finally {
    betSubmitting.value = false
  }
}

const logout = () => {
  window.clearInterval(countdownTimer)
  stopAutoRefresh()
  localStorage.removeItem(STORAGE_KEY)
  localStorage.removeItem(AGREEMENT_KEY)
  form.username = ''
  form.password = ''
  loginResult.value = null
  currentUser.value = null
  games.value = []
  dashboardError.value = ''
  errorMessage.value = ''
  successMessage.value = ''
  view.value = 'login'
}

onMounted(async () => {
  if (loginResult.value && localStorage.getItem(AGREEMENT_KEY) === '1') {
    view.value = 'dashboard'
    activeQuickAction.value = '游戏投注'
    await loadDashboard()
    startAutoRefresh()
  }
})

onBeforeUnmount(() => {
  window.clearInterval(countdownTimer)
  stopAutoRefresh()
})
</script>

<template>
  <main v-if="view === 'login'" class="login-page">
    <section class="login-shell">
      <aside class="brand-panel" aria-label="品牌名称">
        <span>联</span>
        <span>想</span>
      </aside>

      <section class="form-panel">
        <div class="form-badge" aria-hidden="true">
          <span>欢</span>
          <span>迎</span>
          <span>您</span>
        </div>

        <form class="member-form" @submit.prevent="handleSubmit">
          <h1>会 员 登 录</h1>

          <label class="form-row">
            <span>账 号:</span>
            <input v-model="form.username" type="text" autocomplete="username" />
          </label>

          <label class="form-row">
            <span>密 码:</span>
            <input
              v-model="form.password"
              type="password"
              autocomplete="current-password"
            />
          </label>

          <button type="submit" :disabled="loading">
            {{ loading ? '登录中' : '登 录' }}
          </button>

          <p v-if="errorMessage" class="status-message is-error">{{ errorMessage }}</p>
          <p v-else-if="successMessage" class="status-message is-success">
            {{ successMessage }}
          </p>

          <p>Copyrighted 2020 v3.6.0 版权所有</p>
        </form>
      </section>
    </section>
  </main>

  <main v-else-if="view === 'agreement'" class="agreement-page">
    <section class="agreement-shell">
      <div class="agreement-bar">
        <button type="button" @click="agreeProtocol">同意</button>
        <button type="button" @click="rejectProtocol">不同意</button>
      </div>

      <section class="agreement-card">
        <h1>用户协议</h1>

        <div class="agreement-content">
          <p
            v-for="(item, index) in agreementItems"
            :key="item"
            :class="{ 'agreement-warning': index === 3 || index >= 11 }"
          >
            {{ item }}
          </p>
        </div>

        <p class="agreement-copy">Copyright© 2023 版权所有</p>
      </section>

      <div class="agreement-bar">
        <button type="button" @click="agreeProtocol">同意</button>
        <button type="button" @click="rejectProtocol">不同意</button>
      </div>
    </section>
  </main>

  <main v-else class="console-page">
    <header class="console-topbar">
      <div class="console-brand">麒麟盛世</div>
    </header>

    <section class="console-announcement">
      <strong>公告：</strong>
      <span>{{ noticeText }}</span>
    </section>

    <section class="console-layout">
      <aside class="console-sidebar">
        <div class="user-panel">
          <div class="user-row">
            <span>账号</span>
            <strong>{{ currentUser?.username || displayName }}</strong>
          </div>
          <div class="user-row">
            <span>快彩额度</span>
            <strong>{{ currentUser?.balance ?? '--' }}</strong>
          </div>
          <div class="user-row">
            <span>状态</span>
            <strong>在线</strong>
          </div>
          <div class="user-row">
            <span>剩余期数</span>
            <strong>{{ gamePage.total }}</strong>
          </div>
        </div>

        <div class="quick-links">
          <button
            v-for="action in quickActions"
            :key="action"
            type="button"
            :class="{ active: activeQuickAction === action }"
            @click="selectQuickAction(action)"
          >
            {{ action }}
          </button>
        </div>

        <div class="sidebar-actions">
          <button type="button" @click="refreshDashboard">刷新</button>
          <button type="button" @click="backToAgreement">协议</button>
          <button type="button" @click="logout">退出</button>
        </div>

        <section class="bet-slip-panel sidebar-bet-slip">
          <div class="bet-slip-title">确认注单</div>

          <div class="bet-slip-table">
            <div class="bet-slip-head">明细</div>
            <div class="bet-slip-head">赔率</div>
            <div class="bet-slip-head">金额</div>
            <div class="bet-slip-head bet-slip-head-action"></div>

            <template v-if="betSlipItems.length">
              <template v-for="item in betSlipItems" :key="item.playName">
                <div class="bet-slip-cell bet-slip-detail">{{ item.playName }}</div>
                <div class="bet-slip-cell">{{ item.odds }}</div>
                <div class="bet-slip-cell">
                  <input
                    class="bet-slip-input"
                    :value="item.betAmount"
                    type="number"
                    min="0"
                    step="0.01"
                    placeholder="请输入金额"
                    @input="updateBetSlipAmount(item.playName, $event.target.value)"
                  />
                </div>
                <div class="bet-slip-cell bet-slip-action-cell">
                  <button
                    type="button"
                    class="bet-slip-remove"
                    @click="removeBetSlipItem(item.playName)"
                  >
                    x
                  </button>
                </div>
              </template>
            </template>
            <template v-else>
              <div class="bet-slip-cell bet-slip-detail">--</div>
              <div class="bet-slip-cell">--</div>
              <div class="bet-slip-cell">--</div>
              <div class="bet-slip-cell bet-slip-action-cell">
                <button type="button" class="bet-slip-remove" disabled>x</button>
              </div>
            </template>

            <div class="bet-slip-cell bet-slip-label">金额：</div>
            <div class="bet-slip-cell bet-slip-span-3 bet-slip-total-amount">{{ betTotalAmount }}</div>

            <div class="bet-slip-cell bet-slip-summary">笔数：{{ betCount }}</div>
            <div class="bet-slip-cell bet-slip-summary bet-slip-span-3">总额：{{ betTotalAmount }}</div>

            <div class="bet-slip-cell bet-slip-game">{{ currentGamePageTitle }}</div>
            <div class="bet-slip-cell bet-slip-submit-wrap bet-slip-span-3">
              <button
                type="button"
                class="bet-slip-submit"
                :disabled="betSubmitting || !selectedBetPlay"
                @click="submitBet"
              >
                {{ betSubmitting ? '提交中...' : betSubmitButtonText }}
              </button>
            </div>
          </div>

          <p v-if="betSubmitError" class="bet-slip-message is-error">{{ betSubmitError }}</p>
          <p v-else-if="betSubmitMessage" class="bet-slip-message is-success">
            {{ betSubmitMessage }}
          </p>
        </section>

        <div class="bill-card">
          <h3>最近注单</h3>

          <p v-if="betsError" class="bill-message is-error">{{ betsError }}</p>
          <p v-else-if="betsLoading" class="bill-message">下注明细加载中...</p>

          <template v-else>
            <div class="bill-table">
              <div class="bill-head">下注类型</div>
              <div class="bill-head">球位</div>
              <div class="bill-head">玩法</div>
              <div class="bill-head">金额</div>
              <template v-if="displayedBets.length">
                <template v-for="bet in displayedBets" :key="bet.orderNo">
                  <div class="bill-cell bill-type-cell">
                    <span class="bill-type-name">{{ bet.betType || '--' }}</span>
                    <span class="bill-type-issue">{{ bet.issueNo || '--' }}</span>
                  </div>
                  <div class="bill-cell">{{ formatBallIndexLabel(bet.ballIndex) }}</div>
                  <div class="bill-cell">{{ bet.playName || bet.selectionValue || '--' }}</div>
                  <div class="bill-cell">{{ bet.betAmount ?? '--' }}</div>
                </template>
              </template>
              <template v-else>
                <div class="bill-cell bill-empty">--</div>
                <div class="bill-cell bill-empty">暂无数据</div>
                <div class="bill-cell bill-empty">--</div>
                <div class="bill-cell bill-empty">--</div>
              </template>
            </div>

            <div class="bill-pagination">
              <button
                type="button"
                :disabled="betsPage.page <= 0"
                @click="changeBetsPage(betsPage.page - 1)"
              >
                上一页
              </button>
              <span>{{ (betsPage.page || 0) + 1 }} / {{ betsPage.totalPages || 1 }}</span>
              <button
                type="button"
                :disabled="betsPage.totalPages > 0 && betsPage.page >= betsPage.totalPages - 1"
                @click="changeBetsPage(betsPage.page + 1)"
              >
                下一页
              </button>
            </div>
          </template>
        </div>
      </aside>

      <section class="console-main">
        <section v-if="activeQuickAction === '下注明细'" class="bets-view">
          <div class="bets-view-header">
            <h2>下注明细</h2>
          </div>

          <p v-if="betsError" class="console-message is-error">{{ betsError }}</p>
          <p v-else-if="betsLoading" class="console-message">下注明细加载中...</p>

          <div v-else class="bets-table">
            <div class="bets-head">订单号</div>
            <div class="bets-head">下注时间</div>
            <div class="bets-head">下注类型</div>
            <div class="bets-head">球位</div>
            <div class="bets-head">玩法</div>
            <div class="bets-head">赔率</div>
            <div class="bets-head">开奖结果说明</div>
            <div class="bets-head">金额</div>
            <div class="bets-head">有效金额</div>
            <div class="bets-head">派彩</div>
            <div class="bets-head">退水</div>
            <div class="bets-head">结果金额</div>
            <div class="bets-head">状态</div>

            <template v-if="displayedBets.length">
              <template v-for="bet in displayedBets" :key="bet.orderNo">
                <div class="bets-cell">{{ bet.orderNo || '--' }}</div>
                <div class="bets-cell">{{ formatDateTime(bet.createdAt) }}</div>
                <div class="bets-cell bill-type-cell">
                  <span class="bill-type-name">{{ bet.betType || '--' }}</span>
                  <span class="bill-type-issue">{{ bet.issueNo || '--' }}</span>
                </div>
                <div class="bets-cell">{{ formatBallIndexLabel(bet.ballIndex) }}</div>
                <div class="bets-cell">{{ bet.playName || '--' }}</div>
                <div class="bets-cell">{{ getBetOddsDisplay(bet) }}</div>
                <div class="bets-cell bets-result-cell">
                  <template v-if="getBetResultDisplay(bet)">
                    <div class="bets-result-card">
                      <div class="bets-result-title">{{ getBetResultDisplay(bet).title }}</div>
                      <div class="bets-result-row">
                        <span class="bets-result-label">结果：</span>
                        <span
                          v-for="(num, index) in getBetResultDisplay(bet).numbers"
                          :key="`${bet.orderNo || 'bet'}-${index}-${num}`"
                          class="bets-result-number"
                          :class="{ active: index === getBetResultDisplay(bet).highlightIndex }"
                        >
                          {{ num }}
                        </span>
                        <span v-if="getBetResultDisplay(bet).tail" class="bets-result-tail">
                          {{ getBetResultDisplay(bet).tail }}
                        </span>
                      </div>
                    </div>
                  </template>
                  <template v-else>{{ bet.resultDetail || '--' }}</template>
                </div>
                <div class="bets-cell">{{ bet.betAmount ?? '--' }}</div>
                <div class="bets-cell">{{ bet.validAmount ?? '--' }}</div>
                <div class="bets-cell">{{ bet.payoutAmount ?? '--' }}</div>
                <div class="bets-cell">{{ bet.rebateAmount ?? '--' }}</div>
                <div class="bets-cell">{{ bet.resultAmount ?? '--' }}</div>
                <div class="bets-cell">{{ bet.status || '--' }}</div>
              </template>
            </template>
            <template v-else>
              <div class="bets-empty">暂无下注明细</div>
            </template>
          </div>

          <div v-if="!betsLoading && !betsError" class="bets-view-pagination bets-view-pagination-footer">
            <button
              type="button"
              :disabled="betsPage.page <= 0"
              @click="changeBetsPage(betsPage.page - 1)"
            >
              上一页
            </button>
            <span>{{ (betsPage.page || 0) + 1 }} / {{ betsPage.totalPages || 1 }}</span>
            <button
              type="button"
              :disabled="betsPage.totalPages > 0 && betsPage.page >= betsPage.totalPages - 1"
              @click="changeBetsPage(betsPage.page + 1)"
            >
              下一页
            </button>
          </div>
        </section>

        <template v-else>
          <p v-if="dashboardError" class="console-message is-error">{{ dashboardError }}</p>
          <p v-else-if="dashboardLoading" class="console-message">控制台数据加载中...</p>
          <component
            v-else
            :is="currentGamePageComponent"
            :games="games"
            :selected-game="selectedGame"
            :game-page="gamePage"
            :latest-draw-issue-label="latestDrawIssueLabel"
            :latest-draw-numbers="latestDrawNumbers"
            :latest-draw-special="latestDrawSpecial"
            :latest-draw-result-id="displayWinResult?.id || 'draw'"
            :draw-results-error="drawResultsError"
            :draw-results-loading="drawResultsLoading"
            :win-results-error="winResultsError"
            :win-results-loading="winResultsLoading"
            :available-play-modes="availablePlayModes"
            :selected-play-mode="selectedPlayMode"
            :current-play-mode-label="currentPlayModeLabel"
            :current-issue-status-text="currentIssueStatusText"
            :play-settings-error="playSettingsError"
            :play-settings-loading="playSettingsLoading"
            :selected-bet-play="selectedBetPlay"
            :selected-bet-plays="betSlipItems.map((item) => item.playName)"
            :trend-rows="trendRows"
            :selected-ball="selectedBall"
            :ball-options="ballOptions"
            :page-title="currentGamePageTitle"
            :format-date-time="formatDateTime"
            :get-play-odds="getPlayOdds"
            :get-corner-tokens="getCornerTokens"
            @select-game="selectGame"
            @change-play-mode="changePlayMode"
            @select-bet-play="selectBetPlay"
            @select-ball="selectBall"
          />
        </template>
      </section>
    </section>
  </main>
</template>
