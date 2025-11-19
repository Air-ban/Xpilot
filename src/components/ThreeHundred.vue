// Xpilot Copyright 2025 [Fyisvia Virell] — https://mj.fyisvia.com
// Licensed under AGPL-3.0 with Additional Terms (see LICENSE).
// Note: Certain non-code assets (including datasets, content sets, or media files)
// are excluded from the AGPL license and may NOT be publicly published or redistributed
// without written permission from the author. (See LICENSE for details)

<template>
  <ul class="list bg-base-100 sm:rounded-box sm:shadow-md w-[100%] px-2 sm:px-8">
    <li aria-hidden="true" role="presentation" class="p-0 m-0 sm:h-4"></li>

    <li class="p-4 pb-2 text-lg font-semibold opacity-100 tracking-wide">
      {{ t('threeHundred.title') }}
    </li>
    <li aria-hidden="true" role="presentation" class="p-0 m-0 h-2"></li>

    <li class="p-4 pb-0 text-sm sm:text-base opacity-80 tracking-wide">
      <div class="flex flex-col sm:flex-row sm:items-center sm:gap-8">
        <div class="flex-1">
          <div class="badge badge-neutral mr-2">{{ t('threeHundred.labels.no') }} {{ currentQuestion?.no }}</div>
          <span v-if="currentQuestion?.round" class="opacity-80">{{ currentQuestion.round }}</span>
        </div>
        <div class="flex-1 mt-2 sm:mt-0">
          <span class="opacity-80" v-if="currentQuestion?.dora">宝牌指示牌：{{ currentQuestion.dora }}</span>
        </div>
      </div>
    </li>

    <li class="p-4 pb-2 opacity-100 tracking-wide text-base sm:text-lg font-semibold" :style="contentWidthStyle">
      <span>{{ t('efficiencyTrain.ui.handTitle') }}</span>
    </li>

    <li class="p-2 sm:p-4 pb-2 text-xs sm:text-sm md:text-base opacity-100 tracking-wide" :style="contentWidthStyle">
      <div
        class="grid w-full"
        :style="{ gridTemplateColumns: `repeat(${handTiles.length || 1}, 1fr)`, gap: '0px' }"
      >
        <template v-for="(tile, index) in handTiles" :key="tile + '-' + index">
          <img
            :src="tileSrc(tile)"
            :alt="tile"
            @click="handleTileClick(tile)"
            @mouseenter="hoveredIndex = index"
            @mouseleave="hoveredIndex = null"
            class="tile-img transition-transform duration-150 cursor-pointer"
            :style="{ borderRadius: '5px', transform: hoveredIndex === index ? 'translateY(-5px)' : 'none' }"
          />
        </template>
      </div>
    </li>

    <li class="p-4 pb-2 text-xs md:text-base opacity-80 tracking-wide flex items-center gap-4 mx-2">
      <div v-if="!isSmallScreen" class="flex items-center gap-3">
        <input type="range" min="40" max="100" step="5" v-model.number="contentWidth" class="range range-xs w-36 sm:w-40" :aria-label="t('threeHundred.ui.clickHint')" />
        <span class="opacity-80">{{ contentWidthDisplay }}%</span>
      </div>
      <div class="ml-auto">{{ t('threeHundred.ui.clickHint') }}</div>
    </li>

    <li v-if="lastDiscardResult" class="p-2" :style="contentWidthStyle">
      <div class="flex flex-col items-center gap-3">
        <div :class="['inline-flex items-center gap-1 text-lg font-bold', lastDiscardResult.isCorrect ? 'badge badge-success' : 'badge badge-error']">
          <svg v-if="lastDiscardResult.isCorrect" class="size-[1em]" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><g fill="currentColor" stroke-linejoin="miter" stroke-linecap="butt"><circle cx="12" cy="12" r="10" fill="none" stroke="currentColor" stroke-linecap="square" stroke-miterlimit="10" stroke-width="2"></circle><polyline points="7 13 10 16 17 8" fill="none" stroke="currentColor" stroke-linecap="square" stroke-miterlimit="10" stroke-width="2"></polyline></g></svg>
          <svg v-else class="size-[1em]" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><g fill="currentColor"><rect x="1.972" y="11" width="20.056" height="2" transform="translate(-4.971 12) rotate(-45)" fill="currentColor" stroke-width="0"></rect><path d="m12,23c-6.065,0-11-4.935-11-11S5.935,1,12,1s11,4.935,11,11-4.935,11-11,11Zm0-20C7.038,3,3,7.037,3,12s4.038,9,9,9,9-4.037,9-9S16.962,3,12,3Z" stroke-width="0" fill="currentColor"></path></g></svg>
          {{ lastDiscardResult.isCorrect ? t('threeHundred.result.correct') : t('threeHundred.result.wrong') }}
        </div>
        <div class="flex flex-col items-center gap-2 w-full" v-if="lastDiscardResult.correctChoices && lastDiscardResult.correctChoices.length">
          <div class="text-base opacity-80">{{ t('efficiencyTrain.result.correctAnswersTitle') }}</div>
          <div class="flex flex-wrap justify-center items-center w-full" style="gap: 0;">
            <div v-for="tile in lastDiscardResult.correctChoices" :key="'correct-' + tile" class="flex justify-center" :style="{ maxWidth: 'calc(100% / 13.5)' }">
              <img :src="tileSrc(tile)" :alt="tile" class="tile-img" />
            </div>
          </div>
        </div>
      </div>
    </li>

    <li class="list-row flex flex-row justify-between items-center gap-2 w-full">
      <button class="btn btn-sm text-sm sm:text-base px-4" :disabled="currentIndex === 0" @click="prevQuestion">{{ t('threeHundred.buttons.prev') }}</button>
      <button class="btn btn-sm text-sm sm:text-base px-4" @click="nextQuestion">{{ t('threeHundred.buttons.next') }}</button>
    </li>

    <li class="p-4">
      <div class="flex items-center gap-2 pb-4">
        <div class="collapse collapse-arrow bg-base-100 border-base-300 border">
          <input type="checkbox" v-model="showResult" @change="handleAnalysisToggle" />
          <div class="collapse-title text-base sm:text-lg font-semibold text-center pl-12">{{ t('threeHundred.analysis.title') }}</div>
          <div class="collapse-content text-sm sm:text-base md:text-lg mb-0">
            <div class="overflow-x-auto">
              <div class="responsive-table-wrapper hidden sm:block">
                <table class="table table-sm w-full bg-base-100 rounded-lg">
                  <thead>
                    <tr>
                      <th class="text-center">{{ t('threeHundred.table.cut') }}</th>
                      <th class="text-center">{{ t('threeHundred.table.improvements') }}</th>
                      <th class="text-center">{{ t('threeHundred.table.goodShapeRate') }}</th>
                      <th class="text-center">{{ t('threeHundred.table.total') }}</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="([tile, result], idx) in sortedImprovementResults" :key="tile" :class="idx % 2 === 1 ? 'hover:bg-base-300' : ''">
                      <td class="font-bold text-center">{{ tile }}</td>
                      <td class="text-center">{{ formatImprovements(result.improvements) }}</td>
                      <td class="font-bold text-center">{{ result.goodShapeRate.toFixed(0) }}%</td>
                      <td class="font-bold text-center">{{ result.totalCount }}</td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <div class="sm:hidden">
                <div class="responsive-table-wrapper">
                  <table class="table table-xs w-full bg-base-100 rounded-lg">
                    <thead>
                      <tr>
                        <th class="text-center">{{ t('threeHundred.table.cut') }}</th>
                        <th class="text-center">{{ t('threeHundred.table.goodShapeRate') }}</th>
                        <th class="text-center">{{ t('threeHundred.table.total') }}</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="([tile, result], idx) in sortedImprovementResults" :key="'sm-' + tile" :class="idx % 2 === 1 ? 'hover:bg-base-300' : ''">
                        <td class="font-bold text-center">{{ tile || '—' }}</td>
                        <td class="font-bold text-center">{{ result.goodShapeRate.toFixed(0) }}%</td>
                        <td class="font-bold text-center">{{ result.totalCount }}</td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </li>
  </ul>
  
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { Shanten } from '../utils/shanten'
import rawMD from '../../300问.md?raw'

const { t } = useI18n()

const tileSrc = (tile) => `/mahjongfiles/${tile}.png`

const handTiles = ref([])
const tilesStr = ref('')
const tiles34Arr = ref([])
const shantenNum = ref(null)
const hoveredIndex = ref(null)
const improvementResults = ref({})
const showResult = ref(false)
const lastDiscardResult = ref(null)

const questions = ref([])
const currentIndex = ref(0)

const currentQuestion = computed(() => questions.value[currentIndex.value] || null)

const TYPE_ORDER = { m: 0, p: 1, s: 2, z: 3 }
const numVal = (n) => (n === '0' ? 5.5 : +n)
const calcShanten = (arr) => new Shanten().calculateShanten(arr)

const tileWidthPercent = computed(() => {
  const count = Math.max(handTiles.value.length, 1)
  return `${100 / count}%`
})

const sortTiles = (tiles) => {
  return [...tiles].sort((a, b) => {
    const ta = a.slice(-1), tb = b.slice(-1)
    if (TYPE_ORDER[ta] !== TYPE_ORDER[tb]) return TYPE_ORDER[ta] - TYPE_ORDER[tb]
    const na = numVal(a[0]), nb = numVal(b[0])
    return na - nb
  })
}

const convertToTilesStr = (tiles) => {
  const grouped = { m: [], p: [], s: [], z: [] }
  for (const t of tiles) {
    const tp = t.slice(-1)
    const n = t[0]
    grouped[tp].push(n)
  }
  let res = ''
  for (const tp of ['m', 'p', 's', 'z']) {
    if (!grouped[tp].length) continue
    grouped[tp].sort((a, b) => numVal(a) - numVal(b))
    res += grouped[tp].join('') + tp
  }
  return res
}

const convertToTiles34Arr = (tiles) => {
  const arr = Array(34).fill(0)
  for (const t of tiles) {
    const n = t[0] === '0' ? 5 : +t[0]
    const tp = t.slice(-1)
    let idx
    if (tp === 'm') idx = n - 1
    else if (tp === 'p') idx = 9 + (n - 1)
    else if (tp === 's') idx = 18 + (n - 1)
    else if (tp === 'z') idx = 27 + (n - 1)
    if (idx !== undefined) arr[idx]++
  }
  return arr
}

const buildHandCountMap = (hand) => {
  const cnt = {}
  for (const t of hand) cnt[t] = (cnt[t] || 0) + 1
  return cnt
}

const buildCountMap = (tiles) => {
  const map = {}
  for (const t of tiles) map[t] = (map[t] || 0) + 1
  return map
}

const buildSuitTiles = (type) => {
  const tiles = []
  for (let n = 1; n <= 9; n++) {
    if (n === 5) {
      tiles.push(...Array(3).fill(`5${type}`), `0${type}`)
    } else {
      tiles.push(...Array(4).fill(`${n}${type}`))
    }
  }
  return tiles
}
const generateTiles = (mode) => {
  const suits = ['m', 'p', 's']
  const tiles = []
  for (const type of suits) tiles.push(...buildSuitTiles(type))
  if (mode === 'withHonor') {
    for (let n = 1; n <= 7; n++) tiles.push(...Array(4).fill(`${n}z`))
  }
  return tiles
}

const baseCountMap = ref({})
const goodShapeCache = new Map()
const uniqueTiles = computed(() => Object.keys(baseCountMap.value))

const remainingCounts = (handCount) => {
  const res = {}
  for (const tile of uniqueTiles.value) {
    const left = (baseCountMap.value[tile] || 0) - (handCount[tile] || 0)
    if (left > 0) res[tile] = left
  }
  return res
}

const MAX_GOOD_SHAPE_DEPTH = 3
const calculateGoodShapeRate = (hand, arr34, depth = 0) => {
  const key = arr34.join(',') + '|' + depth
  if (goodShapeCache.has(key)) return goodShapeCache.get(key)
  if (depth > MAX_GOOD_SHAPE_DEPTH) return 0
  const shanten = calcShanten(arr34)
  if (shanten === 0) {
    const handCount = buildHandCountMap(hand)
    const rem = remainingCounts(handCount)
    const waitTypes = new Set()
    let waitCount = 0
    for (const [tile, count] of Object.entries(rem)) {
      const tempHand = [...hand, tile]
      const temp34 = convertToTiles34Arr(tempHand)
      if (calcShanten(temp34) === -1) {
        waitTypes.add(tile)
        waitCount += count
      }
    }
    const res = (waitTypes.size > 1 && waitCount > 4) ? 1 : 0
    goodShapeCache.set(key, res)
    return res
  }
  const handCount = buildHandCountMap(hand)
  const rem = remainingCounts(handCount)
  let total = 0
  let good = 0
  for (const [tile, count] of Object.entries(rem)) {
    const tempHand = [...hand, tile]
    const temp34 = convertToTiles34Arr(tempHand)
    const newShanten = calcShanten(temp34)
    if (newShanten < shanten) {
      let bestPath = 0
      const seen = new Set()
      for (let i = 0; i < tempHand.length; i++) {
        const d = tempHand[i]
        if (seen.has(d)) continue
        seen.add(d)
        const discardHand = tempHand.slice()
        discardHand.splice(i, 1)
        const discard34 = convertToTiles34Arr(discardHand)
        if (calcShanten(discard34) < shanten) {
          bestPath = Math.max(bestPath, calculateGoodShapeRate(discardHand, discard34, depth + 1))
        }
      }
      total += count
      good += count * bestPath
    }
  }
  const res = total === 0 ? 0 : good / total
  goodShapeCache.set(key, res)
  return res
}

const analyzeImprovement = (currentHand, current34) => {
  const results = {}
  const originalShanten = calcShanten(current34)
  const globalHandCount = buildHandCountMap(currentHand)
  currentHand.forEach((tileToDiscard, discardIndex) => {
    const newHand = currentHand.filter((_, i) => i !== discardIndex)
    const newHandCount = { ...globalHandCount }
    newHandCount[tileToDiscard]--
    if (newHandCount[tileToDiscard] === 0) delete newHandCount[tileToDiscard]
    const new34 = convertToTiles34Arr(newHand)
    const rem = remainingCounts(newHandCount)
    const improvements = {}
    let totalCount = 0
    for (const [tile, count] of Object.entries(rem)) {
      const tempHand = [...newHand, tile]
      const temp34 = convertToTiles34Arr(tempHand)
      if (calcShanten(temp34) < originalShanten) {
        improvements[tile] = count
        totalCount += count
      }
    }
    if (totalCount > 0) {
      const goodShapeRate = Math.max(0, calculateGoodShapeRate(newHand, new34) * 100)
      results[tileToDiscard] = { improvements, totalCount, goodShapeRate }
    }
  })
  return results
}

const updateTilesState = (tiles) => {
  tilesStr.value = convertToTilesStr(tiles)
  tiles34Arr.value = convertToTiles34Arr(tiles)
  shantenNum.value = calcShanten(tiles34Arr.value)
}

const recalcImprovementResults = (hand = handTiles.value) => {
  improvementResults.value = analyzeImprovement(hand, convertToTiles34Arr(hand))
}

const sortedImprovementResults = computed(() => {
  const entries = Object.entries(improvementResults.value || {})
  return entries.sort(
    (a, b) => ((b[1].totalCount || 0) - (a[1].totalCount || 0)) || ((b[1].goodShapeRate || 0) - (a[1].goodShapeRate || 0))
  )
})

const formatImprovements = (improvements) => sortTiles(Object.keys(improvements)).join(', ')

const setQuestion = (q) => {
  baseCountMap.value = buildCountMap(generateTiles('withHonor'))
  const hand13 = q.hand || []
  const sorted13 = sortTiles(hand13.slice(0, 13))
  const newHand = [...sorted13, q.draw].filter(Boolean)
  handTiles.value = newHand
  showResult.value = false
  lastDiscardResult.value = null
  updateTilesState(newHand)
  recalcImprovementResults(newHand)
}

const handleTileClick = (clickedTile) => {
  const answers = new Set(currentQuestion.value?.answers || [])
  const isCorrect = answers.has(clickedTile)
  lastDiscardResult.value = {
    isCorrect,
    correctChoices: Array.from(answers)
  }
  showResult.value = true
}

const nextQuestion = () => {
  if (currentIndex.value < questions.value.length - 1) {
    currentIndex.value += 1
  } else {
    currentIndex.value = Math.min(currentIndex.value + 1, questions.value.length - 1)
  }
  if (questions.value[currentIndex.value]) setQuestion(questions.value[currentIndex.value])
}

const prevQuestion = () => {
  if (currentIndex.value > 0) currentIndex.value -= 1
  if (questions.value[currentIndex.value]) setQuestion(questions.value[currentIndex.value])
}

const parseTiles = (s) => {
  return (s || '').match(/(?:\d|0)[mpsz]/g) || []
}

const parseMarkdown = (raw) => {
  const lines = raw.split(/\r?\n/)
  const res = []
  let cur = null
  for (const line of lines) {
    if (/^#\s*Q\d+/.test(line)) {
      if (cur) res.push(cur)
      const no = (line.match(/Q(\d+)/) || [])[1] || ''
      cur = { no, round: '', dora: '', hand: [], draw: '', answers: [] }
    } else if (cur && /^-\s*/.test(line)) {
      const content = line.replace(/^-\s*/, '')
      if (content.includes('手牌形状')) {
        const m = content.split('：')[1] || ''
        cur.hand = parseTiles(m)
      } else if (content.includes('摸到')) {
        const m = content.split('：')[1] || ''
        const tiles = parseTiles(m)
        cur.draw = tiles[0] || ''
      } else if (content.includes('答案')) {
        const m = content.split('：')[1] || ''
        cur.answers = parseTiles(m)
      } else if (content.includes('宝牌指示牌')) {
        const m = content.split('：')[1] || ''
        cur.dora = m.trim()
      } else if (/风|东|南|西|北|巡/.test(content)) {
        cur.round = content.trim()
      }
    }
  }
  if (cur) res.push(cur)
  return res.filter(q => q.hand.length >= 13 && q.draw && q.answers.length)
}

const WIDTH_STORAGE_KEY = 'threehundred-content-width'
const WIDTH_CHANGE_EVENT = 'threehundred-content-width-change'
const clampWidth = (value) => Math.min(100, Math.max(40, Number.isFinite(value) ? value : 100))
const getSavedContentWidth = () => {
  try {
    const stored = parseInt(localStorage.getItem(WIDTH_STORAGE_KEY), 10)
    return clampWidth(Number.isFinite(stored) ? stored : 100)
  } catch {
    return 100
  }
}
const isSmallScreen = ref(false)
const contentWidth = ref(getSavedContentWidth())
let widthUpdateFromEvent = false
const contentWidthStyle = computed(() => ({ width: `${isSmallScreen.value ? 100 : contentWidth.value}%`, marginLeft: 'auto', marginRight: 'auto' }))
const contentWidthDisplay = computed(() => (isSmallScreen.value ? 100 : contentWidth.value))

const updateScreenSize = () => { isSmallScreen.value = window.innerWidth < 640 }
const handleWidthBroadcast = (event) => {
  if (typeof event.detail !== 'number') return
  const clamped = clampWidth(event.detail)
  if (clamped === contentWidth.value) return
  widthUpdateFromEvent = true
  contentWidth.value = clamped
}

onMounted(() => {
  questions.value = parseMarkdown(rawMD)
  currentIndex.value = 0
  if (questions.value.length) setQuestion(questions.value[0])
  updateScreenSize()
  window.addEventListener('resize', updateScreenSize)
  window.addEventListener(WIDTH_CHANGE_EVENT, handleWidthBroadcast)
})
onUnmounted(() => {
  window.removeEventListener('resize', updateScreenSize)
  window.removeEventListener(WIDTH_CHANGE_EVENT, handleWidthBroadcast)
})

const handleAnalysisToggle = () => { if (showResult.value) recalcImprovementResults() }
</script>

<style scoped>
.tile-img { width: 100%; height: auto; object-fit: contain; }
</style>

