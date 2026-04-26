<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref, watch } from 'vue'
import Game1Page from './components/Game1Page.vue'
import Game2Page from './components/Game2Page.vue'
import Game3Page from './components/Game3Page.vue'
import Game4Page from './components/Game4Page.vue'
import Game5Page from './components/Game5Page.vue'
import Game6Page from './components/Game6Page.vue'
import Game7Page from './components/Game7Page.vue'

const STORAGE_KEY = 'member-login-session'
const AGREEMENT_KEY = 'member-agreement-accepted'
const DASHBOARD_CACHE_KEY = 'member-dashboard-cache'

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

const quickActions = ['游戏投注', '下注明细', '期数结果', '账户历史', '个人资讯', '游戏规则']
const gameRuleSections = [
  {
    id: 'australia-fantan',
    title: '澳洲八番摊',
    intro:
      '本平台以澳洲幸运八 官方开奖结果为准，开奖结果第八球号码所构成的自然数总和除以4的余数，作为番摊的开奖号码。余数为1是1番、余数为2是2番、余数为3是3番、余数为0是4番。比如澳洲幸运八第八球开奖结果为 5，则计算 5 ÷ 4 = 1 …… 1，其余数为1，即开奖号码为1番。',
    website: 'https://auluckylottery.com',
    plays: [
      '单双：开出13为单，开出24为双',
      '番：彩池“X番”，开出X为赢，其他为输',
      '念：彩池“X念Y”，开出X为赢，开出Y为和，开出其他为输',
      '角：彩池“XY角”，开出X或者Y时为赢，其他为输',
      '正：彩池“X正”，开出X时为赢，开出X的对面为输，开出X的两边为和；13互为对面，24互为对面',
      '中：彩池“XYZ中”，开出X/Y/Z其中一个时为赢，其他为输',
      '通：彩池“YZX通”，X的意思为开出遇到“通”的数字X则输，开出Y或者Z时为赢，其他为和',
      '加：彩池“Y加ZX”，Y的意思为开出为Y数字则赢，开出Z或者X时为和，其他为输',
    ],
    extraTitle: '单边特码规则说明',
    extras: [
      '号码：指下注的特码与开出之号码第八球开奖号码相同，视为中奖，如第八球开出号码 8，下注第八球为 8 者视为中奖，其余情形视为不中奖。',
      '两面：指单、双；大、小。',
      '单、双：号码为双数叫双，如8、16；号码为单数叫单，如19、5。',
      '大、小：开出之号码大于或等于11为大，小于或等于10为小。',
    ],
  },
  {
    id: 'australia-fantan-ball8',
    title: '澳洲八番摊8球',
    intro:
      '本平台以澳洲幸运八 官方开奖结果为准，开奖结果每个球位号码所构成的自然数除以4的余数，作为番摊的开奖号码。余数为1是1番、余数为2是2番、余数为3是3番、余数为0是4番。比如澳洲幸运八第八球开奖结果为 5，则计算 5 ÷ 4 = 1 …… 1，其余数为1，即开奖号码为1番。',
    website: 'https://auluckylottery.com',
    plays: [
      '单双：开出13为单，开出24为双',
      '番：彩池“X番”，开出X为赢，其他为输',
      '念：彩池“X念Y”，开出X为赢，开出Y为和，开出其他为输',
      '角：彩池“XY角”，开出X或者Y时为赢，其他为输',
      '正：彩池“X正”，开出X时为赢，开出X的对面为输，开出X的两边为和；13互为对面，24互为对面',
      '中：彩池“XYZ中”，开出X/Y/Z其中一个时为赢，其他为输',
      '通：彩池“YZX通”，X的意思为开出遇到“通”的数字X则输，开出Y或者Z时为赢，其他为和',
      '加：彩池“Y加ZX”，Y的意思为开出为Y数字则赢，开出Z或者X时为和，其他为输',
    ],
    extraTitle: '单边特码规则说明',
    extras: [
      '号码：指下注的特码与所选球位的开奖号码相同，视为中奖，如第八球开出号码 8，下注第八球为 8 者视为中奖，其余情形视为不中奖。',
      '两面：指单、双；大、小。',
      '单、双：号码为双数叫双，如8、16；号码为单数叫单，如19、5。',
      '大、小：开出之号码大于或等于11为大，小于或等于10为小。',
    ],
  },
  {
    id: 'australia-fantan-1to8',
    title: '澳洲八番摊1到8球',
    intro:
      '本平台以澳洲幸运八 官方开奖结果为准，开奖结果每个球位号码所构成的自然数除以4的余数，作为番摊的开奖号码。余数为1是1番、余数为2是2番、余数为3是3番、余数为0是4番。比如澳洲幸运八第八球开奖结果为 5，则计算 5 ÷ 4 = 1 …… 1，其余数为1，即开奖号码为1番。',
    website: 'https://auluckylottery.com',
    plays: [
      '单双：开出13为单，开出24为双',
      '番：彩池“X番”，开出X为赢，其他为输',
      '念：彩池“X念Y”，开出X为赢，开出Y为和，开出其他为输',
      '角：彩池“XY角”，开出X或者Y时为赢，其他为输',
      '正：彩池“X正”，开出X时为赢，开出X的对面为输，开出X的两边为和；13互为对面，24互为对面',
      '中：彩池“XYZ中”，开出X/Y/Z其中一个时为赢，其他为输',
      '通：彩池“YZX通”，X的意思为开出遇到“通”的数字X则输，开出Y或者Z时为赢，其他为和',
      '加：彩池“Y加ZX”，Y的意思为开出为Y数字则赢，开出Z或者X时为和，其他为输',
    ],
    extraTitle: '单边特码规则说明',
    extras: [
      '号码：指下注的特码与所选球位的开奖号码相同，视为中奖，如第八球开出号码 8，下注第八球为 8 者视为中奖，其余情形视为不中奖。',
      '两面：指单、双；大、小。',
      '单、双：号码为双数叫双，如8、16；号码为单数叫单，如19、5。',
      '大、小：开出之号码大于或等于11为大，小于或等于10为小。',
    ],
  },
  {
    id: 'canada-28',
    title: '加拿大28',
    intro:
      '本平台以加拿大28官方开奖结果为准。例如：开奖号码为1，2，3。开奖结果后两位号码所构成的自然数（即2,3=23）除以4的余数为3，作为【加拿大28番摊后2】的开奖结果【3番】。开奖结果以余数为1是【1番】、余数为2是【2番】、余数为3是【3番】、余数为0是【4番】。',
    website: 'http://jndclub.com/#/Home',
    plays: [
      '单双：开出13为单，开出24为双',
      '番：彩池“X番”，开出X为赢，其他为输',
      '念：彩池“X念Y”，开出X为赢，开出Y为和，开出其他为输',
      '角：彩池“XY角”，开出X或者Y时为赢，其他为输',
      '正：彩池“X正”，开出X时为赢，开出X的对面为输，开出X的两边为和；13互为对面，24互为对面',
      '中：彩池“XYZ中”，开出X/Y/Z其中一个时为赢，其他为输',
      '通：彩池“YZX通”，X的意思为开出遇到“通”的数字X则输，开出Y或者Z时为赢，其他为和',
      '加：彩池“Y加ZX”，Y的意思为开出为Y数字则赢，开出Z或者X时为和，其他为输',
    ],
    extraTitle: '',
    extras: [],
  },
]
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
  '24一通',
  '14二通',
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

const readDashboardCache = () => {
  try {
    const raw = localStorage.getItem(DASHBOARD_CACHE_KEY)
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

const getPayloadMessage = (payload, fallback = '接口请求失败') => {
  if (payload?.data && typeof payload.data === 'object' && payload.data.message) {
    return payload.data.message
  }
  return payload?.message || fallback
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
const manualRefreshLoading = ref(false)
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
const announcements = ref([])
const bets = ref([])
const betsLoading = ref(false)
const betsError = ref('')
const betsFilters = reactive({
  gameId: null,
  startAt: '',
  endAt: '',
  status: '',
})
const betsPage = ref({
  total: 0,
  page: 0,
  size: 6,
  totalPages: 0,
})
const issueResults = ref([])
const issueResultsLoading = ref(false)
const issueResultsError = ref('')
const issueResultsFilters = reactive({
  issueNo: '',
  drawDate: '',
})
const issueResultsPage = ref({
  total: 0,
  page: 0,
  size: 20,
  totalPages: 0,
})
const dailySummary = ref([])
const dailySummaryLoading = ref(false)
const dailySummaryError = ref('')
const dailySummarySelectedDate = ref('')
const dailySummaryGames = ref([])
const dailySummaryGamesLoading = ref(false)
const dailySummaryGamesError = ref('')
const dailySummaryPage = ref({
  total: 0,
  page: 0,
  size: 10,
  totalPages: 0,
})
const dailySummaryFilters = reactive({
  startDate: '',
  endDate: '',
})
const memberInfoTab = ref('profile')
const selectedRuleSectionId = ref(gameRuleSections[0]?.id || '')
const memberInfoPlaySettings = ref([])
const memberInfoLoading = ref(false)
const memberInfoError = ref('')
const memberBalanceLogs = ref([])
const memberBalanceLogsLoading = ref(false)
const memberBalanceLogsError = ref('')
const memberBalanceLogsPage = ref({
  total: 0,
  page: 0,
  size: 10,
  totalPages: 0,
})
const hoveredDailySummaryDate = ref('')
const hoveredDailySummaryGameKey = ref('')
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
let clockTimer = null
let autoRefreshInProgress = false
let autoRefreshTickCount = 0
let lastDefaultAutoRefreshAt = 0
let winResultsRequestId = 0
let drawResultsRequestId = 0
let betsRequestId = 0
let issueResultsRequestId = 0
let playSettingsRequestId = 0
let winResultsLoadingRequestId = 0
let drawResultsLoadingRequestId = 0
let playSettingsLoadingRequestId = 0
let memberInfoRequestId = 0
let memberBalanceLogsRequestId = 0
let globalLoadingShowTimer = null
let globalLoadingHideTimer = null
let globalLoadingShownAt = 0
const playSettingsPrefetchingGameIds = new Set()

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

const formatDateInputValue = (value) => {
  const date = value instanceof Date ? value : new Date(value)
  if (Number.isNaN(date.getTime())) {
    return ''
  }

  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

const formatAnnouncementCurrentTime = (value = new Date()) => {
  const date = value instanceof Date ? value : new Date(value)
  if (Number.isNaN(date.getTime())) {
    return '--'
  }

  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  const seconds = String(date.getSeconds()).padStart(2, '0')
  return `${year}年${month}月${day}日 ${hours}:${minutes}:${seconds}`
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

const currentBetSideType = computed(() => {
  if (selectedPlayMode.value === 'single') {
    return 'SINGLE'
  }
  if (selectedPlayMode.value === 'double') {
    return 'DOUBLE'
  }
  return ''
})

const displayedBets = computed(() => bets.value)
const displayedDailySummary = computed(() => dailySummary.value)
const displayedDailySummaryGames = computed(() => dailySummaryGames.value)
const currentAnnouncementTime = ref(formatAnnouncementCurrentTime())
const announcementText = computed(() => {
  const texts = announcements.value
    .map((item) => String(item?.content || '').trim())
    .filter(Boolean)

  return texts.length ? texts.join('   ｜   ') : '暂无公告'
})
const gameViewCache = ref({})
const memberDisplayName = computed(() => {
  return (
    currentUser.value?.nickname ||
    loginResult.value?.nickname ||
    currentUser.value?.username ||
    loginResult.value?.username ||
    '--'
  )
})

const memberStatusText = computed(() => {
  const rawStatus =
    currentUser.value?.status ??
    currentUser.value?.accountStatus ??
    currentUser.value?.memberStatus ??
    currentUser.value?.enabled

  if (rawStatus === false) {
    return '停用'
  }

  const normalizedStatus = String(rawStatus ?? '').trim().toUpperCase()
  if (['DISABLED', 'INACTIVE', 'FROZEN', 'LOCKED', '停用', '冻结'].includes(normalizedStatus)) {
    return '停用'
  }

  return '启用'
})

const personalInfoCategoryLabelMap = {
  番: '番',
  '大小,單雙': '大小,单双',
  大小单双: '大小单双',
  中: '中',
  念: '念',
  通: '通',
  加: '加',
  正: '正',
  角: '角',
  特码: '特码',
  '特码大单、小双': '大小单双',
}

const personalInfoCategoryOrder = ['番', '大小,单双', '中', '念', '通', '加', '正', '角', '特码', '大小单双']

const getPersonalInfoCategoryLabel = (playName) => {
  const key = resolvePlayOddsKey(playName)
  return personalInfoCategoryLabelMap[key] || key || String(playName || '').trim() || '--'
}

const formatLimitValue = (value) => {
  if (value === null || value === undefined || value === '') {
    return '--'
  }

  const numericValue = Number(value)
  if (!Number.isFinite(numericValue)) {
    return String(value)
  }

  return Number.isInteger(numericValue) ? String(numericValue) : numericValue.toFixed(2)
}

const memberBalanceLogRows = computed(() => {
  return memberBalanceLogs.value.map((item) => {
    const amountChange = Number(item?.amountChange || 0)
    const balanceAfter = Number(item?.balanceAfter || 0)
    const balanceBefore = Number.isFinite(balanceAfter) && Number.isFinite(amountChange)
      ? balanceAfter - amountChange
      : null

    return {
      ...item,
      balanceBefore,
    }
  })
})

const formatSignedMoney = (value) => {
  const numericValue = Number(value)
  if (!Number.isFinite(numericValue)) {
    return formatMoney(value)
  }

  const formatted = formatMoney(Math.abs(numericValue))
  if (numericValue > 0) {
    return `+${formatted}`
  }
  if (numericValue < 0) {
    return `-${formatted}`
  }
  return formatted
}

const currentRuleSection = computed(() => {
  return (
    gameRuleSections.find((item) => item.id === selectedRuleSectionId.value) ||
    gameRuleSections[0] ||
    null
  )
})

const getIssueResultSummary = (item) => {
  const values = [item?.fanResult, item?.oddEvenResult]
    .map((value) => String(value || '').trim())
    .map((value) => value.replace(/翻/g, '番'))
    .filter(Boolean)

  return values.length ? values.join(' / ') : '--'
}

const displayedIssueResults = computed(() => issueResults.value)
const issueResultsTotalAmount = computed(() => {
  const total = displayedIssueResults.value.reduce((sum, item) => {
    const value = Number(item?.memberResultAmount)
    return sum + (Number.isFinite(value) ? value : 0)
  }, 0)

  return formatMoney(total)
})

const formatTemaResult = (value) => {
  const text = String(value || '').trim()
  const match = text.match(/^(\d{1,2})特$/)
  if (!match) {
    return text || '--'
  }

  return `${String(Number(match[1])).padStart(2, '0')}特`
}

const memberInfoSections = computed(() => {
  const groupedByGame = new Map()

  memberInfoPlaySettings.value.forEach((item) => {
    const gameId = Number(item?.gameId || 0)
    const categoryLabel = getPersonalInfoCategoryLabel(item?.playName || item?.playCode)
    if (!groupedByGame.has(gameId)) {
      groupedByGame.set(gameId, new Map())
    }

    const group = groupedByGame.get(gameId)
    const nextPeriodLimit = Number(item?.periodLimit)
    const nextBetLimit = Number(item?.betLimit)
    const currentRow = group.get(categoryLabel) || {
      label: categoryLabel,
      periodLimit: null,
      betLimit: null,
    }

    if (Number.isFinite(nextPeriodLimit)) {
      currentRow.periodLimit =
        currentRow.periodLimit === null ? nextPeriodLimit : Math.max(currentRow.periodLimit, nextPeriodLimit)
    }
    if (Number.isFinite(nextBetLimit)) {
      currentRow.betLimit = currentRow.betLimit === null ? nextBetLimit : Math.max(currentRow.betLimit, nextBetLimit)
    }

    group.set(categoryLabel, currentRow)
  })

  const orderedGameIds = [
    ...games.value
      .map((item) => Number(item?.id || 0))
      .filter((gameId, index, list) => gameId && list.indexOf(gameId) === index && groupedByGame.has(gameId)),
    ...[...groupedByGame.keys()]
      .filter((gameId) => !games.value.some((item) => Number(item?.id || 0) === gameId))
      .sort((left, right) => left - right),
  ]

  return orderedGameIds
    .map((gameId) => {
      const rowsByCategory = groupedByGame.get(gameId)
      const rows = personalInfoCategoryOrder
        .map((label) => rowsByCategory?.get(label))
        .filter(Boolean)

      if (!rows.length) {
        return null
      }

      const game = games.value.find((item) => Number(item?.id || 0) === gameId)
      return {
        gameId,
        title: game?.gameName || `游戏${gameId}`,
        rows,
      }
    })
    .filter(Boolean)
})

const seedMemberInfoFromCurrentGame = () => {
  if (memberInfoPlaySettings.value.length || !playSettings.value.length) {
    return
  }

  const currentGameId = Number(selectedGameId.value || selectedGame.value?.id || 0)
  if (!currentGameId) {
    return
  }

  memberInfoPlaySettings.value = playSettings.value.map((item) => ({
    ...item,
    gameId: Number(item?.gameId || currentGameId),
  }))
}

const queueMemberInfoPrefetch = () => {
  if (!loginResult.value?.token || memberInfoPlaySettings.value.length || memberInfoLoading.value) {
    return
  }

  window.setTimeout(() => {
    if (view.value !== 'dashboard' || !loginResult.value?.token || memberInfoPlaySettings.value.length) {
      return
    }

    void loadMemberInfo(true)
  }, 0)
}

const cacheGameViewData = (gameId, partial) => {
  const normalizedGameId = Number(gameId || 0)
  if (!normalizedGameId) {
    return
  }

  gameViewCache.value = {
    ...gameViewCache.value,
    [normalizedGameId]: {
      ...(gameViewCache.value[normalizedGameId] || {}),
      ...partial,
    },
  }
}

const applyCachedGameViewData = (gameId) => {
  const normalizedGameId = Number(gameId || 0)
  const cached = gameViewCache.value[normalizedGameId]
  if (!cached) {
    return false
  }

  if (Array.isArray(cached.drawResults)) {
    drawResults.value = cached.drawResults
  }
  if (Array.isArray(cached.winResults)) {
    winResults.value = cached.winResults
  }
  if (Array.isArray(cached.playSettings)) {
    playSettings.value = cached.playSettings
  }
  return true
}

const hasCachedPlaySettings = (gameId) => {
  const normalizedGameId = Number(gameId || 0)
  const cachedPlaySettings = gameViewCache.value[normalizedGameId]?.playSettings
  return Array.isArray(cachedPlaySettings) && cachedPlaySettings.length > 0
}

const prefetchPlaySettingsForGame = async (gameId) => {
  const normalizedGameId = Number(gameId || 0)
  if (
    !normalizedGameId ||
    !getAuthToken() ||
    hasCachedPlaySettings(normalizedGameId) ||
    playSettingsPrefetchingGameIds.has(normalizedGameId)
  ) {
    return
  }

  playSettingsPrefetchingGameIds.add(normalizedGameId)

  try {
    const payload = await apiFetch(`/api/member/game-play-settings?gameId=${normalizedGameId}&page=0&size=50`)
    const pageData = payload.data || payload
    const settingsList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    cacheGameViewData(normalizedGameId, {
      playSettings: Array.isArray(settingsList) ? settingsList : [],
    })
    saveDashboardCache()
  } catch {
    // Ignore background prefetch failures and fall back to on-demand loading.
  } finally {
    playSettingsPrefetchingGameIds.delete(normalizedGameId)
  }
}

const queuePlaySettingsPrefetch = (excludeGameId = null) => {
  window.setTimeout(async () => {
    if (view.value !== 'dashboard' || !getAuthToken()) {
      return
    }

    const excludedGameId = Number(excludeGameId || 0)
    const gameIds = games.value
      .map((item) => Number(item?.id || 0))
      .filter((gameId) => gameId && gameId !== excludedGameId && !hasCachedPlaySettings(gameId))

    for (const gameId of gameIds) {
      if (view.value !== 'dashboard' || !getAuthToken()) {
        return
      }

      await prefetchPlaySettingsForGame(gameId)
    }
  }, 0)
}

const saveDashboardCache = () => {
  if (!loginResult.value?.token) {
    return
  }

  const snapshot = {
    userId: loginResult.value.userId ?? null,
    currentUser: currentUser.value,
    games: games.value,
    announcements: announcements.value,
    gamePage: gamePage.value,
    selectedGameId: selectedGameId.value,
    selectedBall: selectedBall.value,
    drawResults: drawResults.value,
    winResults: winResults.value,
    playSettings: playSettings.value,
    gameViewCache: gameViewCache.value,
    memberInfoPlaySettings: memberInfoPlaySettings.value,
    memberBalanceLogs: memberBalanceLogs.value,
    memberBalanceLogsPage: memberBalanceLogsPage.value,
    bets: bets.value,
    betsPage: betsPage.value,
  }

  try {
    localStorage.setItem(DASHBOARD_CACHE_KEY, JSON.stringify(snapshot))
  } catch {
    // Ignore cache write failures so the UI stays responsive.
  }
}

const restoreDashboardCache = () => {
  const snapshot = readDashboardCache()
  if (!snapshot || snapshot.userId !== (loginResult.value?.userId ?? null)) {
    return false
  }

  currentUser.value = snapshot.currentUser || null
  games.value = Array.isArray(snapshot.games) ? snapshot.games : []
  announcements.value = Array.isArray(snapshot.announcements) ? snapshot.announcements : []
  gamePage.value = snapshot.gamePage || gamePage.value
  selectedGameId.value =
    snapshot.selectedGameId && snapshot.games?.some((item) => item.id === snapshot.selectedGameId)
      ? snapshot.selectedGameId
      : snapshot.games?.[0]?.id || null
  drawResults.value = Array.isArray(snapshot.drawResults) ? snapshot.drawResults : []
  winResults.value = Array.isArray(snapshot.winResults) ? snapshot.winResults : []
  playSettings.value = Array.isArray(snapshot.playSettings) ? snapshot.playSettings : []
  gameViewCache.value = snapshot.gameViewCache && typeof snapshot.gameViewCache === 'object'
    ? snapshot.gameViewCache
    : {}
  memberInfoPlaySettings.value = Array.isArray(snapshot.memberInfoPlaySettings)
    ? snapshot.memberInfoPlaySettings
    : []
  memberBalanceLogs.value = Array.isArray(snapshot.memberBalanceLogs) ? snapshot.memberBalanceLogs : []
  memberBalanceLogsPage.value = snapshot.memberBalanceLogsPage || memberBalanceLogsPage.value
  bets.value = Array.isArray(snapshot.bets) ? snapshot.bets : []
  betsPage.value = snapshot.betsPage || betsPage.value

  const supportedBalls = gameBallConfig[Number(selectedGameId.value)] || [1]
  selectedBall.value = supportedBalls.includes(snapshot.selectedBall)
    ? snapshot.selectedBall
    : supportedBalls[0]
  startCountdown(selectedGame.value?.sealCountdownSeconds)
  return true
}

const queueRecentBetsRefresh = (gameId, page = 0) => {
  if (!gameId) {
    return
  }

  window.setTimeout(() => {
    if (
      view.value !== 'dashboard' ||
      activeQuickAction.value !== '游戏投注' ||
      Number(selectedGameId.value) !== Number(gameId)
    ) {
      return
    }

    void loadBets(gameId, page, true)
  }, 0)
}

const GLOBAL_LOADING_DELAY_MS = 280
const GLOBAL_LOADING_MIN_VISIBLE_MS = 420
const GAME_BETTING_REFRESH_INTERVAL_MS = 2000
const DEFAULT_AUTO_REFRESH_INTERVAL_MS = 5000
const DASHBOARD_SHELL_REFRESH_EVERY = 5
const visibleGlobalLoadingText = ref('')
const globalLoadingText = computed(() => {
  if (dashboardLoading.value && !games.value.length) {
    return '资料加载中...'
  }
  if (manualRefreshLoading.value) {
    return '刷新中...'
  }
  if (dailySummarySelectedDate.value && dailySummaryGamesLoading.value) {
    return '明细加载中...'
  }
  if (dailySummaryLoading.value) {
    return '账户历史加载中...'
  }
  if (betsLoading.value) {
    return '下注明细加载中...'
  }
  return ''
})

const clearGlobalLoadingTimers = () => {
  window.clearTimeout(globalLoadingShowTimer)
  window.clearTimeout(globalLoadingHideTimer)
  globalLoadingShowTimer = null
  globalLoadingHideTimer = null
}

watch(
  globalLoadingText,
  (nextText) => {
    window.clearTimeout(globalLoadingHideTimer)
    globalLoadingHideTimer = null

    if (nextText) {
      if (visibleGlobalLoadingText.value) {
        visibleGlobalLoadingText.value = nextText
        return
      }

      window.clearTimeout(globalLoadingShowTimer)
      globalLoadingShowTimer = window.setTimeout(() => {
        visibleGlobalLoadingText.value = nextText
        globalLoadingShownAt = Date.now()
        globalLoadingShowTimer = null
      }, GLOBAL_LOADING_DELAY_MS)
      return
    }

    window.clearTimeout(globalLoadingShowTimer)
    globalLoadingShowTimer = null

    if (!visibleGlobalLoadingText.value) {
      return
    }

    const elapsed = Date.now() - globalLoadingShownAt
    const remaining = Math.max(0, GLOBAL_LOADING_MIN_VISIBLE_MS - elapsed)
    globalLoadingHideTimer = window.setTimeout(() => {
      visibleGlobalLoadingText.value = ''
      globalLoadingShownAt = 0
      globalLoadingHideTimer = null
    }, remaining)
  },
  { immediate: true },
)

const betsPageTotals = computed(() => {
  return displayedBets.value.reduce(
    (accumulator, item) => {
      accumulator.count += 1
      accumulator.betAmount += Number(item?.betAmount) || 0
      accumulator.validAmount += Number(item?.validAmount) || 0
      accumulator.payoutAmount += Number(item?.payoutAmount) || 0
      accumulator.rebateAmount += Number(item?.rebateAmount) || 0
      accumulator.resultAmount += Number(item?.resultAmount) || 0
      return accumulator
    },
    {
      count: 0,
      betAmount: 0,
      validAmount: 0,
      payoutAmount: 0,
      rebateAmount: 0,
      resultAmount: 0,
    },
  )
})
const dailySummaryTotals = computed(() => {
  return displayedDailySummary.value.reduce(
    (accumulator, item) => {
      accumulator.orderCount += Number(item?.orderCount) || 0
      accumulator.betAmount += Number(item?.betAmount) || 0
      accumulator.validAmount += Number(item?.validAmount) || 0
      accumulator.payoutAmount += Number(item?.payoutAmount) || 0
      accumulator.rebateAmount += Number(item?.rebateAmount) || 0
      accumulator.resultAmount += Number(item?.resultAmount) || 0
      return accumulator
    },
    {
      orderCount: 0,
      betAmount: 0,
      validAmount: 0,
      payoutAmount: 0,
      rebateAmount: 0,
      resultAmount: 0,
    },
  )
})
const dailySummaryGamesTotals = computed(() => {
  return displayedDailySummaryGames.value.reduce(
    (accumulator, item) => {
      accumulator.orderCount += Number(item?.orderCount) || 0
      accumulator.betAmount += Number(item?.betAmount) || 0
      accumulator.validAmount += Number(item?.validAmount) || 0
      accumulator.payoutAmount += Number(item?.payoutAmount) || 0
      accumulator.rebateAmount += Number(item?.rebateAmount) || 0
      accumulator.resultAmount += Number(item?.resultAmount) || 0
      return accumulator
    },
    {
      orderCount: 0,
      betAmount: 0,
      validAmount: 0,
      payoutAmount: 0,
      rebateAmount: 0,
      resultAmount: 0,
    },
  )
})

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

const hasLatestDrawDisplayData = computed(() => {
  return Boolean(displayWinResult.value?.drawCode)
})

const isDrawDisplayLoading = computed(() => {
  return (drawResultsLoading.value || winResultsLoading.value) && !hasLatestDrawDisplayData.value
})

const latestDrawNumbers = computed(() => {
  if (isDrawDisplayLoading.value) {
    return ['加载中']
  }

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

const latestDrawActiveIndices = computed(() => {
  const numberCount = latestDrawNumbers.value.length
  if (!numberCount || (numberCount === 1 && latestDrawNumbers.value[0] === '加载中')) {
    return []
  }

  if ([4, 5, 6].includes(Number(selectedGame.value?.id || 0))) {
    return [Math.max(0, numberCount - 2), numberCount - 1]
  }

  return [numberCount - 1]
})

const getHighlightedResultIndices = (gameId, numbers, ballIndex) => {
  const numberCount = Array.isArray(numbers) ? numbers.length : 0
  if (!numberCount) {
    return []
  }

  if ([4, 5, 6].includes(Number(gameId || 0))) {
    return [Math.max(0, numberCount - 2), numberCount - 1]
  }

  const rawBallIndex = Number(ballIndex)
  return Number.isFinite(rawBallIndex) && rawBallIndex > 0 ? [rawBallIndex - 1] : []
}

const latestDrawSpecial = computed(() => {
  if (isDrawDisplayLoading.value) {
    return '加载中'
  }

  const fanText = String(displayWinResult.value?.fanResult || '').trim()
  const matched = fanText.match(/([1-4])/)
  return matched ? matched[1] : '--'
})

const latestDrawIssueLabel = computed(() => {
  if (isDrawDisplayLoading.value) {
    return '加载中'
  }

  return displayWinResult.value?.issueNo || '--'
})

const currentIssueStatusText = computed(() => {
  if (!selectedGame.value) {
    return '资料加载中...'
  }

  if (selectedGame.value.sealed) {
    return '已封盘'
  }

  return `距离封盘${countdownText.value}`
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
const batchBetAmount = ref('')
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

const formatResultStatusLabel = (resultStatus) => {
  const normalizedStatus = String(resultStatus || '').trim().toUpperCase()
  if (normalizedStatus === 'PENDING') {
    return '未开奖'
  }
  if (normalizedStatus === 'WIN') {
    return '中奖'
  }
  if (normalizedStatus === 'DRAW') {
    return '平'
  }
  if (normalizedStatus === 'LOSE') {
    return '未中奖'
  }
  return resultStatus || '--'
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
  const gameId = Number(bet?.gameId || selectedGame.value?.id || 0)
  const numbers = parseDrawCodeNumbers(bet?.drawCode)
  const tail = extractResultTail(bet?.resultDetail)
  const betDetailText = getBetDetailWithoutIssue(bet)
  const oddsText = getBetOddsDisplay(bet)
  const highlightIndices = getHighlightedResultIndices(gameId, numbers, bet?.ballIndex)

  return {
    detailSummary: `${betDetailText} @ ${oddsText}`,
    resultNumbers: numbers.map((value, index) => ({
      value,
      active: highlightIndices.includes(index),
    })),
    tail,
    hasResult: numbers.length > 0 || Boolean(tail),
  }
}

const trendRows = computed(() => {
  const rowCount = 10
  const columnCount = 12
  const maxCellCount = rowCount * columnCount
  const trailingBlankCount = 4
  const maxFilledCellCount = Math.max(0, maxCellCount - trailingBlankCount)
  const matrix = Array.from({ length: rowCount }, () => Array.from({ length: columnCount }, () => ''))

  const orderedFanValues = [...winResults.value]
    .sort((left, right) => compareIssueNo(left?.issueNo, right?.issueNo))
    .map((item) => String(item?.fanResult || '').match(/([1-4])/))
    .map((match) => (match ? Number(match[1]) : null))
    .filter((value) => value !== null)
    .slice(-maxFilledCellCount)

  orderedFanValues.forEach((value, index) => {
    const rowIndex = index % rowCount
    const colIndex = Math.floor(index / rowCount)
    if (colIndex < columnCount) {
      matrix[rowIndex][colIndex] = value
    }
  })

  return matrix
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

const startAnnouncementClock = () => {
  window.clearInterval(clockTimer)
  currentAnnouncementTime.value = formatAnnouncementCurrentTime()
  clockTimer = window.setInterval(() => {
    currentAnnouncementTime.value = formatAnnouncementCurrentTime()
  }, 1000)
}

const redirectToLogin = (message = '登录已失效，请重新登录') => {
  window.clearInterval(countdownTimer)
  window.clearInterval(dashboardPollTimer)
  window.clearInterval(clockTimer)
  dashboardPollTimer = null
  clockTimer = null
  autoRefreshInProgress = false
  autoRefreshTickCount = 0
  lastDefaultAutoRefreshAt = 0
  window.clearTimeout(globalLoadingShowTimer)
  window.clearTimeout(globalLoadingHideTimer)
  globalLoadingShowTimer = null
  globalLoadingHideTimer = null
  visibleGlobalLoadingText.value = ''
  globalLoadingShownAt = 0

  localStorage.removeItem(STORAGE_KEY)
  localStorage.removeItem(AGREEMENT_KEY)
  localStorage.removeItem(DASHBOARD_CACHE_KEY)

  loginResult.value = null
  currentUser.value = null
  games.value = []
  announcements.value = []
  drawResults.value = []
  drawResultsLoading.value = false
  drawResultsError.value = ''
  winResults.value = []
  winResultsLoading.value = false
  winResultsError.value = ''
  playSettings.value = []
  playSettingsLoading.value = false
  playSettingsError.value = ''
  bets.value = []
  betsLoading.value = false
  betsError.value = ''
  dailySummary.value = []
  dailySummaryLoading.value = false
  dailySummaryError.value = ''
  dailySummaryGames.value = []
  dailySummaryGamesLoading.value = false
  dailySummaryGamesError.value = ''
  issueResults.value = []
  issueResultsLoading.value = false
  issueResultsError.value = ''
  memberInfoPlaySettings.value = []
  memberInfoLoading.value = false
  memberInfoError.value = ''
  memberBalanceLogs.value = []
  memberBalanceLogsLoading.value = false
  memberBalanceLogsError.value = ''
  dashboardLoading.value = false
  dashboardError.value = ''
  manualRefreshLoading.value = false
  backgroundRefreshing.value = false
  betSlipItems.value = []
  selectedBetPlay.value = ''
  batchBetAmount.value = ''
  betSubmitError.value = ''
  betSubmitMessage.value = ''
  betSubmitting.value = false
  successMessage.value = ''
  errorMessage.value = message
  view.value = 'login'
}

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
  const payloadMessage = getPayloadMessage(payload)
  const shouldRedirectToLogin =
    url !== '/api/member/login' &&
    Boolean(token) &&
    (response.status === 401 ||
      response.status === 403 ||
      /未登录|登录失效|重新登录|token|unauthorized|forbidden/i.test(payloadMessage))

  if (shouldRedirectToLogin) {
    redirectToLogin(payloadMessage || '登录已失效，请重新登录')
    throw new Error(payloadMessage || '登录已失效，请重新登录')
  }

  if (
    !response.ok ||
    payload.success === false ||
    (payload?.data && typeof payload.data === 'object' && payload.data.success === false)
  ) {
    throw new Error(payloadMessage)
  }

  return payload
}

const loadAnnouncements = async () => {
  if (!getAuthToken()) {
    announcements.value = []
    return
  }

  try {
    const payload = await apiFetch('/api/member/platform-announcements')
    const list = payload.data || payload.list || []
    announcements.value = Array.isArray(list)
      ? [...list]
          .filter((item) => Number(item?.announcementSide || 0) === 2)
          .sort((left, right) => {
            const sortLeft = Number(left?.sortOrder)
            const sortRight = Number(right?.sortOrder)
            if (sortLeft !== sortRight) {
              return (Number.isFinite(sortLeft) ? sortLeft : 0) - (Number.isFinite(sortRight) ? sortRight : 0)
            }

            const timeLeft = new Date(left?.updatedAt || left?.createdAt || 0).getTime()
            const timeRight = new Date(right?.updatedAt || right?.createdAt || 0).getTime()
            return timeRight - timeLeft
          })
      : []
    saveDashboardCache()
  } catch {
    // Keep cached announcements when the notice request fails.
  }
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

const loadDashboard = async (silent = false, refreshDetails = true) => {
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
    const [currentPayload, gamesPayload, announcementsPayload] = await Promise.all([
      apiFetch('/api/member/current'),
      apiFetch('/api/member/games?page=0&size=12'),
      apiFetch('/api/member/platform-announcements').catch(() => null),
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

    const announcementList = announcementsPayload?.data || announcementsPayload?.list || []
    announcements.value = Array.isArray(announcementList)
      ? [...announcementList]
          .filter((item) => Number(item?.announcementSide || 0) === 2)
          .sort((left, right) => {
            const sortLeft = Number(left?.sortOrder)
            const sortRight = Number(right?.sortOrder)
            if (sortLeft !== sortRight) {
              return (Number.isFinite(sortLeft) ? sortLeft : 0) - (Number.isFinite(sortRight) ? sortRight : 0)
            }

            const timeLeft = new Date(left?.updatedAt || left?.createdAt || 0).getTime()
            const timeRight = new Date(right?.updatedAt || right?.createdAt || 0).getTime()
            return timeRight - timeLeft
          })
      : announcements.value

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
      if (!silent) {
        applyCachedGameViewData(selectedGameId.value)
      }
      startCountdown(selectedGame.value?.sealCountdownSeconds)
      saveDashboardCache()
      const shouldSilentRefreshPlaySettings = hasCachedPlaySettings(selectedGameId.value)
      const detailTasks = refreshDetails
        ? [loadDrawResults(selectedGameId.value, silent), loadWinResults(selectedGameId.value, silent)]
        : []
      if (refreshDetails && (!silent || previousGameId !== selectedGameId.value || !playSettings.value.length)) {
        detailTasks.push(loadPlaySettings(selectedGameId.value, silent || shouldSilentRefreshPlaySettings))
      }

      if (silent) {
        await Promise.all(detailTasks)
        if (activeQuickAction.value === '游戏投注') {
          queueRecentBetsRefresh(selectedGameId.value, betsPage.value.page)
        }
        queueMemberInfoPrefetch()
        queuePlaySettingsPrefetch(selectedGameId.value)
      } else {
        void Promise.all(detailTasks)
        if (activeQuickAction.value === '游戏投注') {
          queueRecentBetsRefresh(selectedGameId.value, 0)
        }
        queueMemberInfoPrefetch()
        queuePlaySettingsPrefetch(selectedGameId.value)
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
      saveDashboardCache()
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

  const requestId = ++playSettingsRequestId

  if (!silent) {
    playSettingsLoadingRequestId = requestId
    playSettingsLoading.value = true
    playSettingsError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/member/game-play-settings?gameId=${gameId}&page=0&size=50`)
    const pageData = payload.data || payload
    const settingsList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    if (requestId !== playSettingsRequestId || Number(selectedGameId.value) !== Number(gameId)) {
      return
    }

    playSettings.value = Array.isArray(settingsList) ? settingsList : []
    cacheGameViewData(gameId, { playSettings: playSettings.value })
    saveDashboardCache()
  } catch (error) {
    if (requestId !== playSettingsRequestId || Number(selectedGameId.value) !== Number(gameId)) {
      return
    }

    if (!silent) {
      playSettings.value = []
      playSettingsError.value =
        error instanceof Error ? error.message : '玩法赔率加载失败'
    }
  } finally {
    if (
      !silent &&
      requestId === playSettingsLoadingRequestId &&
      Number(selectedGameId.value) === Number(gameId)
    ) {
      playSettingsLoading.value = false
    }
  }
}

const loadBets = async (gameId, page = 0, silent = false) => {
  const requestId = ++betsRequestId
  const query = new URLSearchParams({
    page: String(page),
    size: String(betsPage.value.size),
  })

  const targetGameId = gameId ?? betsFilters.gameId
  if (targetGameId) {
    query.set('gameId', String(targetGameId))
  }
  if (betsFilters.startAt) {
    query.set('startAt', betsFilters.startAt)
  }
  if (betsFilters.endAt) {
    query.set('endAt', betsFilters.endAt)
  }
  if (betsFilters.status) {
    query.set('status', betsFilters.status)
  }

  if (gameId && !betsFilters.startAt && !betsFilters.endAt) {
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

    if (requestId !== betsRequestId) {
      return
    }

    bets.value = Array.isArray(betList) ? betList : []
    betsPage.value = {
      total: pageData.total || 0,
      page: pageData.page || 0,
      size: pageData.size || betsPage.value.size,
      totalPages: pageData.totalPages || 0,
    }
    saveDashboardCache()
  } catch (error) {
    if (requestId !== betsRequestId) {
      return
    }

    if (!silent) {
      bets.value = []
      betsError.value = error instanceof Error ? error.message : '下注明细加载失败'
    }
  } finally {
    if (!silent && requestId === betsRequestId) {
      betsLoading.value = false
    }
  }
}

const loadDailySummary = async (gameId, page = 0, silent = false) => {
  const query = new URLSearchParams({
    page: String(page),
    size: String(dailySummaryPage.value.size),
  })

  if (gameId) {
    query.set('gameId', String(gameId))
  }
  if (dailySummaryFilters.startDate) {
    query.set('startDate', dailySummaryFilters.startDate)
  }
  if (dailySummaryFilters.endDate) {
    query.set('endDate', dailySummaryFilters.endDate)
  }

  if (!silent) {
    dailySummaryLoading.value = true
    dailySummaryError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/bets/daily-summary?${query.toString()}`)
    const pageData = payload.data || payload
    const summaryList = pageData.records || pageData.content || pageData.list || pageData.items || []

    dailySummary.value = Array.isArray(summaryList) ? summaryList : []
    dailySummaryPage.value = {
      total: pageData.total || 0,
      page: pageData.page || 0,
      size: pageData.size || dailySummaryPage.value.size,
      totalPages: pageData.totalPages || 0,
    }
  } catch (error) {
    if (!silent) {
      dailySummary.value = []
      dailySummaryError.value = error instanceof Error ? error.message : '账户历史加载失败'
    }
  } finally {
    if (!silent) {
      dailySummaryLoading.value = false
    }
  }
}

const loadDailySummaryGames = async (tradeDate, silent = false) => {
  if (!tradeDate) {
    dailySummaryGames.value = []
    dailySummaryGamesError.value = ''
    return
  }

  if (!silent) {
    dailySummaryGamesLoading.value = true
    dailySummaryGamesError.value = ''
  }

  try {
    const query = new URLSearchParams({
      tradeDate: String(tradeDate),
    })
    const payload = await apiFetch(`/api/bets/daily-summary/games?${query.toString()}`)
    const list = payload.data || payload.list || []
    dailySummaryGames.value = Array.isArray(list) ? list : []
  } catch (error) {
    if (!silent) {
      dailySummaryGames.value = []
      dailySummaryGamesError.value = error instanceof Error ? error.message : '账户历史明细加载失败'
    }
  } finally {
    if (!silent) {
      dailySummaryGamesLoading.value = false
    }
  }
}

const loadMemberInfo = async (silent = false) => {
  const requestId = ++memberInfoRequestId

  if (!silent) {
    memberInfoLoading.value = true
    memberInfoError.value = ''
  }

  try {
    const [currentPayload, settingsPayload] = await Promise.all([
      apiFetch('/api/member/current'),
      apiFetch('/api/member/game-play-settings?page=0&size=200'),
    ])

    if (requestId !== memberInfoRequestId) {
      return
    }

    currentUser.value = currentPayload.data || currentUser.value
    const pageData = settingsPayload.data || settingsPayload
    const settingsList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    memberInfoPlaySettings.value = Array.isArray(settingsList) ? settingsList : []
    saveDashboardCache()
  } catch (error) {
    if (requestId !== memberInfoRequestId) {
      return
    }

    if (!silent) {
      memberInfoPlaySettings.value = []
      memberInfoError.value = error instanceof Error ? error.message : '个人资讯加载失败'
    }
  } finally {
    if (!silent && requestId === memberInfoRequestId) {
      memberInfoLoading.value = false
    }
  }
}

const loadMemberBalanceLogs = async (page = 0, silent = false) => {
  const requestId = ++memberBalanceLogsRequestId
  const query = new URLSearchParams({
    page: String(page),
    size: String(memberBalanceLogsPage.value.size),
  })

  if (!silent) {
    memberBalanceLogsLoading.value = true
    memberBalanceLogsError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/member/balance-logs?${query.toString()}`)
    if (requestId !== memberBalanceLogsRequestId) {
      return
    }

    const pageData = payload.data || payload
    const list = pageData.records || pageData.content || pageData.list || pageData.items || []
    memberBalanceLogs.value = Array.isArray(list) ? list : []
    memberBalanceLogsPage.value = {
      total: pageData.total || 0,
      page: pageData.page || 0,
      size: pageData.size || memberBalanceLogsPage.value.size,
      totalPages: pageData.totalPages || 0,
    }
    saveDashboardCache()
  } catch (error) {
    if (requestId !== memberBalanceLogsRequestId) {
      return
    }

    if (!silent) {
      memberBalanceLogs.value = []
      memberBalanceLogsError.value = error instanceof Error ? error.message : '会员账本加载失败'
    }
  } finally {
    if (!silent && requestId === memberBalanceLogsRequestId) {
      memberBalanceLogsLoading.value = false
    }
  }
}

const loadIssueResults = async (gameId, page = 0, silent = false) => {
  if (!gameId) {
    issueResults.value = []
    issueResultsError.value = ''
    return
  }

  const requestId = ++issueResultsRequestId
  const query = new URLSearchParams({
    gameId: String(gameId),
    page: String(page),
    size: String(issueResultsPage.value.size),
  })

  if (issueResultsFilters.issueNo.trim()) {
    query.set('issueNo', issueResultsFilters.issueNo.trim())
  }

  if (!silent) {
    issueResultsLoading.value = true
    issueResultsError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/member/game-win-results?${query.toString()}`)
    if (requestId !== issueResultsRequestId || Number(selectedGameId.value) !== Number(gameId)) {
      return
    }

    const pageData = payload.data || payload
    const list = pageData.records || pageData.content || pageData.list || pageData.items || []
    issueResults.value = Array.isArray(list) ? list : []
    issueResultsPage.value = {
      total: pageData.total || 0,
      page: pageData.page || 0,
      size: pageData.size || issueResultsPage.value.size,
      totalPages: pageData.totalPages || 0,
    }
  } catch (error) {
    if (requestId !== issueResultsRequestId || Number(selectedGameId.value) !== Number(gameId)) {
      return
    }

    if (!silent) {
      issueResults.value = []
      issueResultsError.value = error instanceof Error ? error.message : '期数结果加载失败'
    }
  } finally {
    if (!silent && requestId === issueResultsRequestId && Number(selectedGameId.value) === Number(gameId)) {
      issueResultsLoading.value = false
    }
  }
}

const loadWinResults = async (gameId, silent = false) => {
  if (!gameId) {
    winResults.value = []
    winResultsError.value = ''
    return
  }

  const requestId = ++winResultsRequestId

  const query = new URLSearchParams({
    gameId: String(gameId),
    page: '0',
    size: '120',
  })

  if (!silent) {
    winResultsLoadingRequestId = requestId
    winResultsLoading.value = true
    winResultsError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/member/game-win-results?${query.toString()}`)
    const pageData = payload.data || payload
    const resultList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    if (requestId !== winResultsRequestId || Number(selectedGameId.value) !== Number(gameId)) {
      return
    }

    winResults.value = Array.isArray(resultList) ? resultList : []
    cacheGameViewData(gameId, { winResults: winResults.value })
    saveDashboardCache()
  } catch (error) {
    if (requestId !== winResultsRequestId || Number(selectedGameId.value) !== Number(gameId)) {
      return
    }

    if (!silent) {
      winResults.value = []
      winResultsError.value =
        error instanceof Error ? error.message : '中奖结果加载失败'
    }
  } finally {
    if (
      !silent &&
      requestId === winResultsLoadingRequestId &&
      Number(selectedGameId.value) === Number(gameId)
    ) {
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

  const requestId = ++drawResultsRequestId

  if (!silent) {
    drawResultsLoadingRequestId = requestId
    drawResultsLoading.value = true
    drawResultsError.value = ''
  }

  try {
    const payload = await apiFetch(`/api/member/game-draw-results?gameId=${gameId}&page=0&size=10`)
    const pageData = payload.data || payload
    const resultList =
      pageData.records || pageData.content || pageData.list || pageData.items || []

    if (requestId !== drawResultsRequestId || Number(selectedGameId.value) !== Number(gameId)) {
      return
    }

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
    cacheGameViewData(gameId, { drawResults: drawResults.value })
    saveDashboardCache()
  } catch (error) {
    if (requestId !== drawResultsRequestId || Number(selectedGameId.value) !== Number(gameId)) {
      return
    }

    if (!silent) {
      drawResults.value = []
      drawResultsError.value =
        error instanceof Error ? error.message : '开奖历史加载失败'
    }
  } finally {
    if (
      !silent &&
      requestId === drawResultsLoadingRequestId &&
      Number(selectedGameId.value) === Number(gameId)
    ) {
      drawResultsLoading.value = false
    }
  }
}

const stopAutoRefresh = () => {
  window.clearInterval(dashboardPollTimer)
  dashboardPollTimer = null
}

const refreshActiveView = async () => {
  if (activeQuickAction.value === '游戏投注') {
    if (!selectedGameId.value) {
      return
    }

    const refreshGameId = selectedGameId.value
    await Promise.all([
      loadDrawResults(refreshGameId, true),
      loadWinResults(refreshGameId, true),
    ])

    autoRefreshTickCount += 1
    if (autoRefreshTickCount % DASHBOARD_SHELL_REFRESH_EVERY === 0) {
      void loadAnnouncements()
      void loadDashboard(true, false)
    }
    return
  }

  if (activeQuickAction.value === '下注明细') {
    return
  }

  if (activeQuickAction.value === '期数结果') {
    if (!selectedGameId.value) {
      return
    }
    await loadIssueResults(selectedGameId.value, issueResultsPage.value.page, true)
    return
  }

  if (activeQuickAction.value === '账户历史') {
    return
  }

  if (activeQuickAction.value === '个人资讯') {
    if (memberInfoTab.value === 'ledger') {
      await Promise.all([loadMemberInfo(true), loadMemberBalanceLogs(memberBalanceLogsPage.value.page, true)])
      return
    }
    await loadMemberInfo(true)
    return
  }

  if (activeQuickAction.value === '游戏规则') {
    return
  }

  if (!selectedGameId.value) {
    return
  }

  void loadAnnouncements()
  await loadDashboard(true)
}

const startAutoRefresh = () => {
  stopAutoRefresh()
  autoRefreshInProgress = false
  autoRefreshTickCount = 0
  lastDefaultAutoRefreshAt = 0

  dashboardPollTimer = window.setInterval(async () => {
    if (
      view.value !== 'dashboard' ||
      !selectedGameId.value ||
      dashboardLoading.value ||
      autoRefreshInProgress
    ) {
      return
    }

    if (activeQuickAction.value !== '游戏投注') {
      const now = Date.now()
      if (now - lastDefaultAutoRefreshAt < DEFAULT_AUTO_REFRESH_INTERVAL_MS) {
        return
      }
      lastDefaultAutoRefreshAt = now
    }

    autoRefreshInProgress = true
    try {
      await refreshActiveView()
    } finally {
      autoRefreshInProgress = false
    }
  }, GAME_BETTING_REFRESH_INTERVAL_MS)
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
    localStorage.removeItem(DASHBOARD_CACHE_KEY)

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

const selectMemberInfoTab = async (tab) => {
  memberInfoTab.value = tab
  if (tab === 'ledger' && !memberBalanceLogs.value.length) {
    await loadMemberBalanceLogs(0, false)
  }
}

const changeMemberBalanceLogsPage = async (nextPage) => {
  if (nextPage < 0) {
    return
  }
  if (memberBalanceLogsPage.value.totalPages > 0 && nextPage >= memberBalanceLogsPage.value.totalPages) {
    return
  }

  await loadMemberBalanceLogs(nextPage, false)
}

const applyIssueResultsFilters = async () => {
  await loadIssueResults(selectedGameId.value, 0, false)
}

const resetIssueResultsFilters = async () => {
  issueResultsFilters.issueNo = ''
  issueResultsFilters.drawDate = ''
  await loadIssueResults(selectedGameId.value, 0, false)
}

const changeIssueResultsPage = async (nextPage) => {
  if (nextPage < 0) {
    return
  }
  if (issueResultsPage.value.totalPages > 0 && nextPage >= issueResultsPage.value.totalPages) {
    return
  }

  await loadIssueResults(selectedGameId.value, nextPage, false)
}

const handleIssueResultsGameChange = async (value) => {
  const nextGameId = Number(value || 0) || null
  if (!nextGameId || Number(selectedGameId.value) === nextGameId) {
    return
  }

  await selectGame(nextGameId)
}

const openGameBetting = async () => {
  activeQuickAction.value = '游戏投注'
  if (selectedGameId.value) {
    await Promise.all([
      loadDrawResults(selectedGameId.value, false),
      loadWinResults(selectedGameId.value, false),
      loadPlaySettings(selectedGameId.value, false),
    ])
    queueRecentBetsRefresh(selectedGameId.value, 0)
  }
}

const selectQuickAction = async (action) => {
  activeQuickAction.value = action
  if (action === '下注明细') {
    resetBetsFilters()
    betsFilters.status = '未结算'
    bets.value = []
    betsError.value = ''
    await loadBets(selectedGameId.value, 0, false)
    return
  }

  if (action === '期数结果') {
    await loadIssueResults(selectedGameId.value, 0, false)
    return
  }

  if (action === '账户历史') {
    ensureDailySummaryDefaultFilters()
    closeDailySummaryDate()
    await loadDailySummary(null, 0, false)
    return
  }

  if (action === '个人资讯') {
    seedMemberInfoFromCurrentGame()
    if (memberInfoPlaySettings.value.length) {
      memberInfoError.value = ''
      if (memberInfoTab.value === 'ledger' && !memberBalanceLogs.value.length) {
        void loadMemberBalanceLogs(0, true)
      }
      void loadMemberInfo(true)
      return
    }
    if (memberInfoTab.value === 'ledger') {
      await Promise.all([loadMemberInfo(false), loadMemberBalanceLogs(0, false)])
      return
    }
    await loadMemberInfo(false)
    return
  }

  if (action === '游戏投注' && selectedGameId.value) {
    resetBetsFilters()
    const shouldSilentRefreshPlaySettings = hasCachedPlaySettings(selectedGameId.value)
    await Promise.all([
      loadDrawResults(selectedGameId.value, false),
      loadWinResults(selectedGameId.value, false),
      loadPlaySettings(selectedGameId.value, shouldSilentRefreshPlaySettings),
    ])
    queueRecentBetsRefresh(selectedGameId.value, 0)
    queuePlaySettingsPrefetch(selectedGameId.value)
  }
}

const selectBetPlay = (playName) => {
  betSubmitError.value = ''
  betSubmitMessage.value = ''

  const existingItem = betSlipItems.value.find((item) => item.playName === playName)
  if (existingItem) {
    removeBetSlipItem(playName)
    return
  }

  selectedBetPlay.value = playName
  const playSetting = findPlaySettingByPlayName(playName)
  betSlipItems.value.push({
    playName,
    odds: getPlayOdds(playName),
    betAmount: batchBetAmount.value,
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
  batchBetAmount.value = ''
  drawResultsError.value = ''
  winResultsError.value = ''
  playSettingsError.value = ''
  applyCachedGameViewData(gameId)
  const shouldSilentRefreshPlaySettings = hasCachedPlaySettings(gameId)

  if (activeQuickAction.value === '下注明细') {
    bets.value = []
    betsError.value = ''
    await loadBets(gameId, 0, false)
    return
  }

  if (activeQuickAction.value === '期数结果') {
    await loadIssueResults(gameId, 0, false)
    return
  }

  if (activeQuickAction.value === '账户历史') {
    if (dailySummarySelectedDate.value) {
      await loadDailySummaryGames(dailySummarySelectedDate.value, false)
      return
    }
    await loadDailySummary(null, 0, false)
    return
  }

  if (activeQuickAction.value === '个人资讯') {
    return
  }

  if (activeQuickAction.value === '游戏规则') {
    return
  }

  await Promise.all([
    loadDrawResults(gameId, false),
    loadWinResults(gameId, false),
    loadPlaySettings(gameId, shouldSilentRefreshPlaySettings),
  ])
  queueRecentBetsRefresh(gameId, 0)
  queuePlaySettingsPrefetch(gameId)
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

const changeDailySummaryPage = async (nextPage) => {
  if (nextPage < 0) {
    return
  }
  if (dailySummaryPage.value.totalPages > 0 && nextPage >= dailySummaryPage.value.totalPages) {
    return
  }

  await loadDailySummary(null, nextPage, false)
}

const resetBetsFilters = () => {
  betsFilters.gameId = null
  betsFilters.startAt = ''
  betsFilters.endAt = ''
  betsFilters.status = ''
}

const openBetsFromDailySummaryGame = async (item) => {
  const targetGameId = Number(item?.gameId || 0) || null
  if (!targetGameId || !dailySummarySelectedDate.value) {
    return
  }

  selectedGameId.value = targetGameId
  betsFilters.gameId = targetGameId
  betsFilters.startAt = `${dailySummarySelectedDate.value}T00:00:00`
  betsFilters.endAt = `${dailySummarySelectedDate.value}T23:59:59`
  betsFilters.status = ''
  bets.value = []
  betsError.value = ''
  activeQuickAction.value = '下注明细'
  await loadBets(null, 0, false)
}

const openDailySummaryDate = async (tradeDate) => {
  dailySummarySelectedDate.value = String(tradeDate || '').trim()
  await loadDailySummaryGames(dailySummarySelectedDate.value, false)
}

const closeDailySummaryDate = () => {
  dailySummarySelectedDate.value = ''
  dailySummaryGames.value = []
  dailySummaryGamesError.value = ''
}

const applyDailySummaryFilters = async () => {
  closeDailySummaryDate()
  await loadDailySummary(null, 0, false)
}

const resetDailySummaryFilters = async () => {
  closeDailySummaryDate()
  dailySummaryFilters.startDate = ''
  dailySummaryFilters.endDate = ''
  await loadDailySummary(null, 0, false)
}

const setDailySummaryThisWeek = async () => {
  const now = new Date()
  const currentDay = now.getDay()
  const mondayOffset = currentDay === 0 ? -6 : 1 - currentDay
  const monday = new Date(now)
  monday.setDate(now.getDate() + mondayOffset)

  dailySummaryFilters.startDate = formatDateInputValue(monday)
  dailySummaryFilters.endDate = formatDateInputValue(now)
  closeDailySummaryDate()
  await loadDailySummary(null, 0, false)
}

const ensureDailySummaryDefaultFilters = () => {
  if (dailySummaryFilters.startDate && dailySummaryFilters.endDate) {
    return
  }

  const now = new Date()
  const currentDay = now.getDay()
  const mondayOffset = currentDay === 0 ? -6 : 1 - currentDay
  const monday = new Date(now)
  monday.setDate(now.getDate() + mondayOffset)

  dailySummaryFilters.startDate = formatDateInputValue(monday)
  dailySummaryFilters.endDate = formatDateInputValue(now)
}

const refreshDashboard = async () => {
  manualRefreshLoading.value = true
  try {
    await refreshActiveView()
  } finally {
    manualRefreshLoading.value = false
  }
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

const syncBatchBetAmount = () => {
  if (!betSlipItems.value.length) {
    batchBetAmount.value = ''
    return
  }

  const [firstItem] = betSlipItems.value
  const firstAmount = String(firstItem?.betAmount ?? '')
  const isSameAmount = betSlipItems.value.every((item) => String(item?.betAmount ?? '') === firstAmount)
  batchBetAmount.value = isSameAmount ? firstAmount : ''
}

const updateBetSlipAmount = (playName, value) => {
  const targetItem = betSlipItems.value.find((item) => item.playName === playName)
  if (!targetItem) {
    return
  }

  targetItem.betAmount = value
  syncBatchBetAmount()
  selectedBetPlay.value = playName
  betSubmitError.value = ''
  betSubmitMessage.value = ''
}

const updateBatchBetAmount = (value) => {
  batchBetAmount.value = value
  betSlipItems.value.forEach((item) => {
    item.betAmount = value
  })
  betSubmitError.value = ''
  betSubmitMessage.value = ''
}

const removeBetSlipItem = (playName) => {
  betSlipItems.value = betSlipItems.value.filter((item) => item.playName !== playName)
  syncBatchBetAmount()
  if (selectedBetPlay.value === playName) {
    selectedBetPlay.value = betSlipItems.value[betSlipItems.value.length - 1]?.playName || ''
  }
  betSubmitError.value = ''
  betSubmitMessage.value = ''
}

const clearBetSlip = () => {
  selectedBetPlay.value = ''
  betSlipItems.value = []
  batchBetAmount.value = ''
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
          ...(currentBetSideType.value ? { betSideType: currentBetSideType.value } : {}),
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
    batchBetAmount.value = ''
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
  window.clearInterval(clockTimer)
  autoRefreshInProgress = false
  autoRefreshTickCount = 0
  lastDefaultAutoRefreshAt = 0
  stopAutoRefresh()
  localStorage.removeItem(STORAGE_KEY)
  localStorage.removeItem(AGREEMENT_KEY)
  localStorage.removeItem(DASHBOARD_CACHE_KEY)
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
  startAnnouncementClock()
  if (loginResult.value && localStorage.getItem(AGREEMENT_KEY) === '1') {
    view.value = 'dashboard'
    activeQuickAction.value = '游戏投注'
    restoreDashboardCache()
    await loadDashboard()
    queueMemberInfoPrefetch()
    startAutoRefresh()
  }
})

onBeforeUnmount(() => {
  window.clearInterval(countdownTimer)
  window.clearInterval(clockTimer)
  autoRefreshInProgress = false
  autoRefreshTickCount = 0
  lastDefaultAutoRefreshAt = 0
  stopAutoRefresh()
  clearGlobalLoadingTimers()
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
      <div v-if="visibleGlobalLoadingText" class="console-global-loading">
        {{ visibleGlobalLoadingText }}
      </div>
    </header>

    <section class="console-announcement">
      <strong class="console-announcement-label">公告：</strong>
      <div class="console-announcement-track">
        <div class="console-announcement-marquee">
          <span>{{ announcementText }}</span>
          <span aria-hidden="true">{{ announcementText }}</span>
        </div>
      </div>
      <div class="console-announcement-time">
        <span>当前时间:</span>
        <strong>{{ currentAnnouncementTime }}</strong>
      </div>
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
            <span>今日输赢</span>
            <strong>{{ formatMoney(currentUser?.todayResultAmount) }}</strong>
          </div>
          <div class="user-row">
            <span>未结算金额</span>
            <strong>{{ formatMoney(currentUser?.unsettledOrderAmount) }}</strong>
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
            <div class="bet-slip-cell bet-slip-span-3">
              <input
                class="bet-slip-input bet-slip-input-wide"
                :value="batchBetAmount"
                type="number"
                min="0"
                step="0.01"
                placeholder="请输入统一金额"
                @input="updateBatchBetAmount($event.target.value)"
              />
            </div>

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
          <p v-else-if="betsLoading && !displayedBets.length" class="bill-message">下注明细加载中...</p>

          <template v-if="!betsError">
            <div class="bill-table">
              <div class="bill-head">游戏</div>
              <div class="bill-head">下注明细</div>
              <div class="bill-head">赔率</div>
              <div class="bill-head">金额</div>
              <template v-if="displayedBets.length">
                <template v-for="bet in displayedBets" :key="bet.orderNo">
                  <div class="bill-cell bill-type-cell">
                    <span class="bill-type-name">{{ bet.gameName || selectedGame?.gameName || '--' }}</span>
                    <span class="bill-type-issue">{{ bet.issueNo || '--' }}</span>
                  </div>
                  <div class="bill-cell">{{ getBetDetailWithoutIssue(bet) }}</div>
                  <div class="bill-cell">{{ getBetOddsDisplay(bet) }}</div>
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
          <p v-else-if="betsLoading && !displayedBets.length" class="console-message">下注明细加载中...</p>

          <div v-if="!betsError" class="bets-table-wrap">
            <div class="bets-table">
              <div class="bets-head">订单号</div>
              <div class="bets-head">下注时间</div>
              <div class="bets-head">下注类型</div>
              <div class="bets-head">开奖结果说明</div>
              <div class="bets-head">金额</div>
              <div class="bets-head">有效金额</div>
              <div class="bets-head">派彩</div>
              <div class="bets-head">退水</div>
              <div class="bets-head">结果金额</div>
              <div class="bets-head">判定状态</div>
              <div class="bets-head">状态</div>

              <template v-if="displayedBets.length">
                <template v-for="bet in displayedBets" :key="bet.orderNo">
                  <div class="bets-cell">{{ bet.orderNo || '--' }}</div>
                  <div class="bets-cell">{{ formatDateTime(bet.createdAt) }}</div>
                  <div class="bets-cell bill-type-cell">
                    <span class="bill-type-name">{{ bet.betType || '--' }}</span>
                    <span class="bill-type-issue">{{ bet.issueNo || '--' }}</span>
                  </div>
                  <div class="bets-cell bets-result-cell">
                    <template v-if="getBetResultDisplay(bet)">
                      <div class="bets-result-card">
                        <div class="bets-result-line">{{ getBetResultDisplay(bet).detailSummary }}</div>
                        <div v-if="getBetResultDisplay(bet).hasResult" class="bets-result-line">
                          <span>结果：</span>
                          <span class="bets-result-numbers">
                            <span
                              v-for="(item, index) in getBetResultDisplay(bet).resultNumbers"
                              :key="`${bet.orderNo || bet.issueNo || 'bet'}-${index}`"
                              :class="['bets-result-number', { 'is-hit': item.active }]"
                            >
                              {{ item.value }}
                            </span>
                          </span>
                          <template v-if="getBetResultDisplay(bet).tail">
                            <span> = </span>
                            <span class="bets-result-tail">{{ getBetResultDisplay(bet).tail }}</span>
                          </template>
                        </div>
                      </div>
                    </template>
                    <template v-else>{{ getBetDetailWithoutIssue(bet) }} @ {{ getBetOddsDisplay(bet) }}</template>
                  </div>
                  <div class="bets-cell">{{ bet.betAmount ?? '--' }}</div>
                  <div class="bets-cell">{{ bet.validAmount ?? '--' }}</div>
                  <div class="bets-cell">{{ bet.payoutAmount ?? '--' }}</div>
                  <div class="bets-cell">{{ bet.rebateAmount ?? '--' }}</div>
                  <div class="bets-cell">{{ bet.resultAmount ?? '--' }}</div>
                  <div class="bets-cell">{{ formatResultStatusLabel(bet.resultStatus) }}</div>
                  <div class="bets-cell">{{ bet.status || '--' }}</div>
                </template>
                <div class="bets-cell history-total-cell">本页统计</div>
                <div class="bets-cell history-total-cell">--</div>
                <div class="bets-cell history-total-cell">--</div>
                <div class="bets-cell history-total-cell">{{ betsPageTotals.count }}笔</div>
                <div class="bets-cell history-total-cell">{{ formatMoney(betsPageTotals.betAmount) }}</div>
                <div class="bets-cell history-total-cell">{{ formatMoney(betsPageTotals.validAmount) }}</div>
                <div class="bets-cell history-total-cell">{{ formatMoney(betsPageTotals.payoutAmount) }}</div>
                <div class="bets-cell history-total-cell">{{ formatMoney(betsPageTotals.rebateAmount) }}</div>
                <div class="bets-cell history-total-cell">{{ formatMoney(betsPageTotals.resultAmount) }}</div>
                <div class="bets-cell history-total-cell">--</div>
                <div class="bets-cell history-total-cell">--</div>
              </template>
              <template v-else>
                <div class="bets-empty">暂无下注明细</div>
              </template>
            </div>
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

        <section v-else-if="activeQuickAction === '期数结果'" class="bets-view">
          <div class="issue-results-toolbar">
            <div class="issue-results-toolbar-title">彩票开奖结果</div>
            <label class="issue-results-toolbar-field">
              <span>彩票类型:</span>
              <select :value="selectedGameId || ''" @change="handleIssueResultsGameChange($event.target.value)">
                <option v-for="game in games" :key="game.id" :value="game.id">
                  {{ game.gameName }}
                </option>
              </select>
            </label>
            <label class="issue-results-toolbar-field">
              <span>期数:</span>
              <input
                v-model="issueResultsFilters.issueNo"
                type="text"
                placeholder="请输入期号"
                @change="applyIssueResultsFilters"
              />
            </label>
            <label class="issue-results-toolbar-field">
              <span>时间:</span>
              <input v-model="issueResultsFilters.drawDate" type="date" />
            </label>
            <button type="button" class="issue-results-toolbar-button" @click="applyIssueResultsFilters">搜索</button>
            <button type="button" class="issue-results-toolbar-button" @click="openGameBetting">返回投注页面</button>
          </div>

          <p v-if="issueResultsError" class="console-message is-error">{{ issueResultsError }}</p>
          <p v-else-if="issueResultsLoading && !displayedIssueResults.length" class="console-message">期数结果加载中...</p>

          <div v-if="!issueResultsError" class="bets-table-wrap">
            <div class="issue-results-table">
              <div class="bets-head">期数</div>
              <div class="bets-head">开奖时间</div>
              <div class="bets-head">开奖号码</div>
              <div class="bets-head issue-results-head-span">开奖结果</div>
              <div class="bets-head issue-results-last-col">期输赢</div>

              <div class="bets-cell issue-results-total-cell">总计：</div>
              <div class="bets-cell issue-results-total-cell">--</div>
              <div class="bets-cell issue-results-total-cell">--</div>
              <div class="bets-cell issue-results-total-cell issue-results-total-span">--</div>
              <div
                class="bets-cell issue-results-total-cell issue-results-amount-cell issue-results-last-col"
                :class="{ 'is-positive': Number(issueResultsTotalAmount) > 0, 'is-negative': Number(issueResultsTotalAmount) < 0 }"
              >
                {{ issueResultsTotalAmount }}
              </div>

              <template v-if="displayedIssueResults.length">
                <template v-for="item in displayedIssueResults" :key="item.id || item.issueNo">
                  <div class="bets-cell">{{ item.issueNo || '--' }}</div>
                  <div class="bets-cell">{{ formatDateTime(item.drawTime) }}</div>
                  <div class="bets-cell issue-results-code-cell">
                    <div class="draw-balls issue-results-balls">
                      <span
                        v-for="(ball, index) in parseDrawCodeNumbers(item.drawCode)"
                        :key="`${item.issueNo || 'issue'}-${index}`"
                        :class="[
                          'draw-ball',
                          'issue-result-ball',
                          {
                            active: getHighlightedResultIndices(
                              item.gameId || selectedGameId,
                              parseDrawCodeNumbers(item.drawCode),
                              item.ballIndex,
                            ).includes(index),
                          },
                        ]"
                      >
                        {{ ball }}
                      </span>
                    </div>
                  </div>
                  <div class="bets-cell issue-results-result-cell issue-results-fan-cell">
                    {{ String(item.fanResult || '--').replace(/翻/g, '番') }}
                  </div>
                  <div class="bets-cell issue-results-result-cell">
                    {{ item.oddEvenResult || '--' }}
                  </div>
                  <div class="bets-cell issue-results-result-cell">
                    {{ Number(item.gameId || selectedGameId) === 4 ? '-' : formatTemaResult(item.temaResult) }}
                  </div>
                  <div
                    class="bets-cell issue-results-amount-cell issue-results-last-col"
                    :class="{ 'is-positive': Number(item.memberResultAmount) > 0, 'is-negative': Number(item.memberResultAmount) < 0 }"
                  >
                    {{ formatMoney(item.memberResultAmount) }}
                  </div>
                </template>
              </template>
              <template v-else>
                <div class="bets-empty">暂无期数结果</div>
              </template>
            </div>
          </div>

          <div v-if="!issueResultsLoading && !issueResultsError" class="bets-view-pagination bets-view-pagination-footer">
            <button
              type="button"
              :disabled="issueResultsPage.page <= 0"
              @click="changeIssueResultsPage(issueResultsPage.page - 1)"
            >
              上一页
            </button>
            <span>{{ (issueResultsPage.page || 0) + 1 }} / {{ issueResultsPage.totalPages || 1 }}</span>
            <button
              type="button"
              :disabled="issueResultsPage.totalPages > 0 && issueResultsPage.page >= issueResultsPage.totalPages - 1"
              @click="changeIssueResultsPage(issueResultsPage.page + 1)"
            >
              下一页
            </button>
          </div>
        </section>

        <section v-else-if="activeQuickAction === '账户历史'" class="bets-view">
          <div class="bets-view-header">
            <h2>账户历史</h2>
          </div>

          <div v-if="!dailySummarySelectedDate" class="history-filters">
            <label class="history-filter-field">
              <span>起：</span>
              <input v-model="dailySummaryFilters.startDate" type="date" @change="applyDailySummaryFilters" />
            </label>
            <label class="history-filter-field">
              <span>止：</span>
              <input v-model="dailySummaryFilters.endDate" type="date" @change="applyDailySummaryFilters" />
            </label>
            <button type="button" class="history-filter-button" @click="resetDailySummaryFilters">重置</button>
            <button type="button" class="history-filter-button" @click="setDailySummaryThisWeek">本周</button>
          </div>

          <div v-else class="history-detail-toolbar">
            <button type="button" class="history-back-button" @click="closeDailySummaryDate">返回</button>
            <span class="history-detail-title">{{ dailySummarySelectedDate }}</span>
          </div>

          <p v-if="dailySummarySelectedDate && dailySummaryGamesError" class="console-message is-error">
            {{ dailySummaryGamesError }}
          </p>
          <p
            v-else-if="dailySummarySelectedDate && dailySummaryGamesLoading && !displayedDailySummaryGames.length"
            class="console-message"
          >
            账户历史明细加载中...
          </p>
          <p v-else-if="dailySummaryError" class="console-message is-error">{{ dailySummaryError }}</p>
          <p v-else-if="dailySummaryLoading && !displayedDailySummary.length" class="console-message">
            账户历史加载中...
          </p>

          <div
            v-if="
              !(dailySummarySelectedDate && dailySummaryGamesError) &&
              !dailySummaryError
            "
            class="bets-table-wrap"
          >
            <template v-if="dailySummarySelectedDate">
              <div class="history-table history-table-detail">
                <div class="bets-head">交易日期</div>
                <div class="bets-head">彩券种类</div>
                <div class="bets-head">注单数</div>
                <div class="bets-head">下注金额</div>
                <div class="bets-head">有效金额</div>
                <div class="bets-head">派彩</div>
                <div class="bets-head">退水</div>
                <div class="bets-head">结果</div>

                <template v-if="displayedDailySummaryGames.length">
                  <template v-for="item in displayedDailySummaryGames" :key="`${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}-detail`">
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryGameKey === `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}` }"
                      @mouseenter="hoveredDailySummaryGameKey = `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}`"
                      @mouseleave="hoveredDailySummaryGameKey = ''"
                      @click="openBetsFromDailySummaryGame(item)"
                    >
                      {{ item.tradeDate || dailySummarySelectedDate }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryGameKey === `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}` }"
                      @mouseenter="hoveredDailySummaryGameKey = `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}`"
                      @mouseleave="hoveredDailySummaryGameKey = ''"
                      @click="openBetsFromDailySummaryGame(item)"
                    >
                      {{ item.betType || '--' }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryGameKey === `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}` }"
                      @mouseenter="hoveredDailySummaryGameKey = `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}`"
                      @mouseleave="hoveredDailySummaryGameKey = ''"
                      @click="openBetsFromDailySummaryGame(item)"
                    >
                      {{ item.orderCount ?? '--' }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryGameKey === `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}` }"
                      @mouseenter="hoveredDailySummaryGameKey = `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}`"
                      @mouseleave="hoveredDailySummaryGameKey = ''"
                      @click="openBetsFromDailySummaryGame(item)"
                    >
                      {{ formatMoney(item.betAmount) }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryGameKey === `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}` }"
                      @mouseenter="hoveredDailySummaryGameKey = `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}`"
                      @mouseleave="hoveredDailySummaryGameKey = ''"
                      @click="openBetsFromDailySummaryGame(item)"
                    >
                      {{ formatMoney(item.validAmount) }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryGameKey === `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}` }"
                      @mouseenter="hoveredDailySummaryGameKey = `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}`"
                      @mouseleave="hoveredDailySummaryGameKey = ''"
                      @click="openBetsFromDailySummaryGame(item)"
                    >
                      {{ formatMoney(item.payoutAmount) }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryGameKey === `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}` }"
                      @mouseenter="hoveredDailySummaryGameKey = `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}`"
                      @mouseleave="hoveredDailySummaryGameKey = ''"
                      @click="openBetsFromDailySummaryGame(item)"
                    >
                      {{ formatMoney(item.rebateAmount) }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryGameKey === `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}` }"
                      @mouseenter="hoveredDailySummaryGameKey = `${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}`"
                      @mouseleave="hoveredDailySummaryGameKey = ''"
                      @click="openBetsFromDailySummaryGame(item)"
                    >
                      {{ formatMoney(item.resultAmount) }}
                    </div>
                  </template>
                  <div class="bets-cell history-total-cell">共计：</div>
                  <div class="bets-cell history-total-cell">--</div>
                  <div class="bets-cell history-total-cell">{{ dailySummaryGamesTotals.orderCount }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryGamesTotals.betAmount) }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryGamesTotals.validAmount) }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryGamesTotals.payoutAmount) }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryGamesTotals.rebateAmount) }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryGamesTotals.resultAmount) }}</div>
                </template>
                <template v-else>
                  <div class="bets-empty">暂无账户历史明细</div>
                </template>
              </div>
            </template>
            <template v-else>
              <div class="history-table">
                <div class="bets-head">交易日期</div>
                <div class="bets-head">游戏</div>
                <div class="bets-head">注单数</div>
                <div class="bets-head">下注金额</div>
                <div class="bets-head">有效金额</div>
                <div class="bets-head">派彩</div>
                <div class="bets-head">退水</div>
                <div class="bets-head">结果金额</div>

                <template v-if="displayedDailySummary.length">
                  <template v-for="item in displayedDailySummary" :key="`${item.tradeDate}-${item.gameId || 'all'}-${item.betType || 'all'}`">
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryDate === item.tradeDate }"
                      @mouseenter="hoveredDailySummaryDate = item.tradeDate"
                      @mouseleave="hoveredDailySummaryDate = ''"
                      @click="openDailySummaryDate(item.tradeDate)"
                    >
                      <span class="history-date-link">{{ item.tradeDate || '--' }}</span>
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryDate === item.tradeDate }"
                      @mouseenter="hoveredDailySummaryDate = item.tradeDate"
                      @mouseleave="hoveredDailySummaryDate = ''"
                      @click="openDailySummaryDate(item.tradeDate)"
                    >
                      {{ item.betType || '全部游戏' }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryDate === item.tradeDate }"
                      @mouseenter="hoveredDailySummaryDate = item.tradeDate"
                      @mouseleave="hoveredDailySummaryDate = ''"
                      @click="openDailySummaryDate(item.tradeDate)"
                    >
                      {{ item.orderCount ?? '--' }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryDate === item.tradeDate }"
                      @mouseenter="hoveredDailySummaryDate = item.tradeDate"
                      @mouseleave="hoveredDailySummaryDate = ''"
                      @click="openDailySummaryDate(item.tradeDate)"
                    >
                      {{ formatMoney(item.betAmount) }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryDate === item.tradeDate }"
                      @mouseenter="hoveredDailySummaryDate = item.tradeDate"
                      @mouseleave="hoveredDailySummaryDate = ''"
                      @click="openDailySummaryDate(item.tradeDate)"
                    >
                      {{ formatMoney(item.validAmount) }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryDate === item.tradeDate }"
                      @mouseenter="hoveredDailySummaryDate = item.tradeDate"
                      @mouseleave="hoveredDailySummaryDate = ''"
                      @click="openDailySummaryDate(item.tradeDate)"
                    >
                      {{ formatMoney(item.payoutAmount) }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryDate === item.tradeDate }"
                      @mouseenter="hoveredDailySummaryDate = item.tradeDate"
                      @mouseleave="hoveredDailySummaryDate = ''"
                      @click="openDailySummaryDate(item.tradeDate)"
                    >
                      {{ formatMoney(item.rebateAmount) }}
                    </div>
                    <div
                      class="bets-cell history-row-link-cell"
                      :class="{ 'history-row-hovered': hoveredDailySummaryDate === item.tradeDate }"
                      @mouseenter="hoveredDailySummaryDate = item.tradeDate"
                      @mouseleave="hoveredDailySummaryDate = ''"
                      @click="openDailySummaryDate(item.tradeDate)"
                    >
                      {{ formatMoney(item.resultAmount) }}
                    </div>
                  </template>
                  <div class="bets-cell history-total-cell">总计</div>
                  <div class="bets-cell history-total-cell">--</div>
                  <div class="bets-cell history-total-cell">{{ dailySummaryTotals.orderCount }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryTotals.betAmount) }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryTotals.validAmount) }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryTotals.payoutAmount) }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryTotals.rebateAmount) }}</div>
                  <div class="bets-cell history-total-cell">{{ formatMoney(dailySummaryTotals.resultAmount) }}</div>
                </template>
                <template v-else>
                  <div class="bets-empty">暂无账户历史</div>
                </template>
              </div>
            </template>
          </div>

          <div
            v-if="!dailySummarySelectedDate && !dailySummaryLoading && !dailySummaryError"
            class="bets-view-pagination bets-view-pagination-footer"
          >
            <button
              type="button"
              :disabled="dailySummaryPage.page <= 0"
              @click="changeDailySummaryPage(dailySummaryPage.page - 1)"
            >
              上一页
            </button>
            <span>{{ (dailySummaryPage.page || 0) + 1 }} / {{ dailySummaryPage.totalPages || 1 }}</span>
            <button
              type="button"
              :disabled="dailySummaryPage.totalPages > 0 && dailySummaryPage.page >= dailySummaryPage.totalPages - 1"
              @click="changeDailySummaryPage(dailySummaryPage.page + 1)"
            >
              下一页
            </button>
          </div>
        </section>

        <section v-else-if="activeQuickAction === '个人资讯'" class="member-info-view">
          <div class="member-info-shell">
            <div class="member-info-top">
              <div class="member-info-tabs">
                <button
                  type="button"
                  class="member-info-tab"
                  :class="{ 'is-active': memberInfoTab === 'profile' }"
                  @click="selectMemberInfoTab('profile')"
                >
                  个人资讯
                </button>
                <button
                  type="button"
                  class="member-info-tab"
                  :class="{ 'is-active': memberInfoTab === 'ledger' }"
                  @click="selectMemberInfoTab('ledger')"
                >
                  会员账本
                </button>
              </div>

              <div class="member-info-summary">
                <div class="member-info-label">账号:</div>
                <div class="member-info-value">{{ currentUser?.username || displayName }}</div>
                <div class="member-info-label">会员名称:</div>
                <div class="member-info-value">{{ memberDisplayName }}</div>

                <div class="member-info-label">可用额度:</div>
                <div class="member-info-value">{{ formatMoney(currentUser?.balance) }}</div>
                <div class="member-info-label">账号状态:</div>
                <div class="member-info-value">{{ memberStatusText }}</div>
              </div>
            </div>

            <template v-if="memberInfoTab === 'profile'">
              <p v-if="memberInfoError" class="console-message is-error">{{ memberInfoError }}</p>
              <p v-else-if="memberInfoLoading && !memberInfoSections.length" class="console-message">
                个人资讯加载中...
              </p>

              <div v-if="!memberInfoError" class="member-info-sections">
                <template v-if="memberInfoSections.length">
                  <section
                    v-for="section in memberInfoSections"
                    :key="section.gameId"
                    class="member-info-card"
                  >
                    <div class="member-info-card-title">{{ section.title }}</div>
                    <div class="member-info-table">
                      <div class="member-info-head">玩法</div>
                      <div class="member-info-head">单注最高</div>
                      <div class="member-info-head">单期限高</div>
                      <div class="member-info-head">退水(%)</div>

                      <template v-for="row in section.rows" :key="`${section.gameId}-${row.label}`">
                        <div class="member-info-cell member-info-play">{{ row.label }}</div>
                        <div class="member-info-cell">{{ formatLimitValue(row.betLimit) }}</div>
                        <div class="member-info-cell">{{ formatLimitValue(row.periodLimit) }}</div>
                        <div class="member-info-cell">0.0%</div>
                      </template>
                    </div>
                  </section>
                </template>
                <div v-else class="bets-empty">暂无个人资讯</div>
              </div>
            </template>

            <template v-else>
              <p v-if="memberBalanceLogsError" class="console-message is-error">{{ memberBalanceLogsError }}</p>
              <p
                v-else-if="memberBalanceLogsLoading && !memberBalanceLogRows.length"
                class="console-message"
              >
                会员账本加载中...
              </p>

              <div v-if="!memberBalanceLogsError" class="member-info-card">
                <div class="member-ledger-table">
                  <div class="member-info-head">冲账额度</div>
                  <div class="member-info-head">操作前额度</div>
                  <div class="member-info-head">操作后额度</div>
                  <div class="member-info-head">时间</div>

                  <template v-if="memberBalanceLogRows.length">
                    <template v-for="item in memberBalanceLogRows" :key="item.id">
                      <div class="member-info-cell member-info-play">{{ formatSignedMoney(item.amountChange) }}</div>
                      <div class="member-info-cell">{{ formatMoney(item.balanceBefore) }}</div>
                      <div class="member-info-cell">{{ formatMoney(item.balanceAfter) }}</div>
                      <div class="member-info-cell member-ledger-time-cell" :title="item.fullRemark || ''">
                        {{ formatDateTime(item.createdAt) }}
                      </div>
                    </template>
                  </template>
                  <div v-else class="bets-empty">暂无会员账本</div>
                </div>
              </div>

              <div
                v-if="!memberBalanceLogsLoading && !memberBalanceLogsError"
                class="bets-view-pagination bets-view-pagination-footer"
              >
                <button
                  type="button"
                  :disabled="memberBalanceLogsPage.page <= 0"
                  @click="changeMemberBalanceLogsPage(memberBalanceLogsPage.page - 1)"
                >
                  上一页
                </button>
                <span>{{ (memberBalanceLogsPage.page || 0) + 1 }} / {{ memberBalanceLogsPage.totalPages || 1 }}</span>
                <button
                  type="button"
                  :disabled="
                    memberBalanceLogsPage.totalPages > 0 &&
                    memberBalanceLogsPage.page >= memberBalanceLogsPage.totalPages - 1
                  "
                  @click="changeMemberBalanceLogsPage(memberBalanceLogsPage.page + 1)"
                >
                  下一页
                </button>
              </div>
            </template>
          </div>
        </section>

        <section v-else-if="activeQuickAction === '游戏规则'" class="rules-view">
          <div class="rules-shell">
            <div class="rules-title-bar">游戏规则</div>

            <div class="rules-tabs">
              <button
                v-for="section in gameRuleSections"
                :key="section.id"
                type="button"
                class="rules-tab"
                :class="{ active: selectedRuleSectionId === section.id }"
                @click="selectedRuleSectionId = section.id"
              >
                {{ section.title }}
              </button>
            </div>

            <div v-if="currentRuleSection" class="rules-content">
              <section class="rules-block">
                <h3>简介</h3>
                <p>{{ currentRuleSection.intro }}</p>
                <p>
                  官网:
                  <a :href="currentRuleSection.website" target="_blank" rel="noreferrer">
                    {{ currentRuleSection.website }}
                  </a>
                </p>
              </section>

              <section class="rules-block">
                <h3>游戏玩法</h3>
                <ul class="rules-list">
                  <li v-for="item in currentRuleSection.plays" :key="item">{{ item }}</li>
                </ul>
              </section>

              <section
                v-if="currentRuleSection.extraTitle && currentRuleSection.extras.length"
                class="rules-block"
              >
                <h3>{{ currentRuleSection.extraTitle }}</h3>
                <ul class="rules-list">
                  <li v-for="item in currentRuleSection.extras" :key="item">{{ item }}</li>
                </ul>
              </section>
            </div>
          </div>
        </section>

        <template v-else>
          <p v-if="dashboardError" class="console-message is-error">{{ dashboardError }}</p>
          <component
            v-if="games.length"
            :is="currentGamePageComponent"
            :games="games"
            :selected-game="selectedGame"
            :game-page="gamePage"
            :latest-draw-issue-label="latestDrawIssueLabel"
            :latest-draw-numbers="latestDrawNumbers"
            :latest-draw-active-indices="latestDrawActiveIndices"
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
