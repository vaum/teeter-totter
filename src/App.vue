<template>
  <div class="page__container">
    <header class="page__header">
      <p class="page__h">header</p>
      <span class="page__btn"
            v-on:click="startFall">
        Fall
      </span>
    </header>
    <section class="game__globalContainer">
      <div class="game__mainContainer">
        <section ref="playArea" class="game__activeContainer">
          <!-- here the block will be inserted -->
        </section>

        <section class="game__passiveContainer">
          <!-- area where user cannot play in -->
        </section>
      </div>
      <div id="paddle" class="block__paddle" data-value="0">
        <div id="pCenter"></div>
      </div>
    </section>
  </div>
</template>

<script>
/* eslint-disable vue/no-unused-components */
import Block from '@/components/basic/Block'
import { createApp } from 'vue'

export default {
  name: 'App',
  components: { Block },
  methods: {
    startGameLoop () {
      this.updatedAt = Date.now()
      requestAnimationFrame(this.loop)
    },

    loop() {
      const currentTime = Date.now()
      const delta = currentTime - this.updatedAt
      this.updatedAt = currentTime

      if (this.isFalling) this.fallStep(delta)

      if (this.isIntersected()) {
        this.isFalling = false
        this.attachToPaddle()

        this.bend(this.calcBend())

        if (this.isPaddleBentToEnd()) {
          location.reload()
          alert('Game ended =)')
        }

        this.createBlock()
      }

      requestAnimationFrame(this.loop)
    },

    createBlock() {
      const MAX_PERCENTAGE = 85
      const tempDiv = document.createElement('div')
      const instance = createApp(Block, {
        figureClassName: this.getBlockRandomClassName(),
        weight: this.getRandomWeight(),
        left: Math.floor(Math.random() * MAX_PERCENTAGE)
      }).mount(tempDiv)

      this.currentBlock = instance.$el
      this.$refs.playArea.appendChild(instance.$el)
    },

    getRandomWeight() {
      return Math.ceil(Math.random() * 10)
    },

    getBlockRandomClassName() {
      const classNames = ['__square', '__circle', '__triangle']
      return classNames[Math.floor(Math.random()*classNames.length)];
    },

    startFall() {
      this.isFalling = true
      this.currentBlockMarginTop = parseInt(getComputedStyle(this.currentBlock).marginTop)
    },

    fallStep(delta) {
      this.currentBlockMarginTop += this.fallSpeed * delta
      this.currentBlock.style.marginTop = this.currentBlockMarginTop + 'px'
    },

    isIntersected() {
      const MINIMUM_DISTANCE = 1 
      let initDist = this.getInitDist()
      let isFirstRun = Boolean(parseFloat(this.paddle.dataset.value) === 0)

      if (isFirstRun) return Boolean(MINIMUM_DISTANCE > initDist)

      return Boolean(MINIMUM_DISTANCE > this.getRemainingDistance())
    },

    getRemainingDistance() {
      let initDist = this.getInitDist()
      let cathetusLeft = this.calcLeftCathetus()
      let cathetusRight = this.calcRightCathetus()

      return initDist + cathetusLeft + cathetusRight;
    },

    getInitDist() {
      return this.paddle.getBoundingClientRect().top - this.currentBlock.getBoundingClientRect().bottom
    },

    calcLeftCathetus() {
      let cathetusLong = parseFloat(getComputedStyle(this.$refs.playArea).width)
          - (parseFloat(getComputedStyle(this.currentBlock).width) + parseFloat(getComputedStyle(this.currentBlock).marginLeft))
      let cosA = Math.cos(parseFloat(this.paddle.dataset.value) * (Math.PI / 180))
      let hip = cathetusLong / cosA

      return Math.sqrt(hip*hip - cathetusLong*cathetusLong)
    },

    calcRightCathetus() {
      let hippoAlt = parseFloat(getComputedStyle(this.paddle).width) / 2.0
      let sinA = Math.sin(parseFloat(this.paddle.dataset.value) * (Math.PI / 180))

      return hippoAlt * sinA
    },

    attachToPaddle() {
      this.splashedCounter++
      this.currentBlock.id = "splashedFigure" + this.splashedCounter
      this.currentBlock = document.getElementById(this.currentBlock.id)
      this.paddle.appendChild(this.currentBlock)
      this.currentBlock.style.marginTop = "0px"
      this.currentBlock.style.left = (parseInt(this.currentBlock.style.marginLeft) / 2) + "%"
      this.currentBlock.style.marginLeft = "0px"
      this.currentBlock.classList.add('figure__landed')
    },

    calcBend() {
      const bendingCoefficient = 500,
            blockWeight = parseInt(this.currentBlock.dataset.weight)
      let currentAngle = parseFloat(this.paddle.dataset.value),
          additionalAngle = 0,
          distanceToCenter = this.getDist(this.currentBlock, this.paddleCenter)

      additionalAngle = (blockWeight * (distanceToCenter / bendingCoefficient)).toFixed(2)

      currentAngle = currentAngle + parseFloat(additionalAngle)
      this.paddle.dataset.value = currentAngle.toString()

      return currentAngle
    },

    bend(deg) {
      this.paddle.style.transform = `rotate(-${deg}deg)`
    },

    isPaddleBentToEnd() {
      const MAX_BEND_DEG = 12
      return MAX_BEND_DEG <= parseFloat(this.paddle.dataset.value)
    },

    getDist(element1, element2) {
      let dx = element1.offsetLeft - element2.offsetLeft,
          dy = element1.offsetTop - element2.offsetTop;
      return Math.sqrt(Math.pow(dx,2) + Math.pow(dy,2))
    },
  },

  mounted() {
    this.paddle = document.getElementById('paddle')
    this.paddleCenter = document.getElementById('pCenter')
    this.loop = this.loop.bind(this)

    this.createBlock()
    this.startGameLoop()
  },

  data() {
    return {
      currentBlock: {},
      paddle: {},
      paddleCenter: {},
      isFalling: false,
      fallIterator: 0,
      fallingBlocks: [],
      splashedCounter: 0,
      updatedAt: 0,
      gameState: '',
      fallSpeed: 0.25
    }
  }
}
</script>

<style lang="scss">
@import "@/styles/app";
</style>
