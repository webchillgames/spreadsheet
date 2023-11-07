<template>
  <div ref="wrapperRef" class="wrapper" id="js-main-wrapper">
    <canvas ref="spreadsheetRef" @click="handleCanvas" :width="canvasWidth"></canvas>
  </div>
</template>

<script>
import { computed, onMounted, ref } from 'vue'

const table = {
  titles: [
    { id: 1, title: 'Задача 1' },
    { id: 2, title: 'Задача 2' }
  ],

  children: [
    {
      id: 1,
      name: 'Kate',
      days: [
        { id: 1, title: 'Задача 1', data: '5' },
        { id: 2, title: 'Задача 2', data: '3' }
      ]
    },
    {
      id: 2,
      name: 'Sid',
      days: [
        { id: 1, title: 'Задача 1', data: '2' },
        { id: 2, title: 'Задача 2', data: '5' }
      ]
    }
  ]
}

const DEFAULT_CELL_WIDTH = 150
const DEFAULT_FONT_SIZE = 18
const DEFAULT_CELL_HEIGHT = DEFAULT_FONT_SIZE + DEFAULT_FONT_SIZE / 2
const X_OF_FIRST_CELL = 0

export default {
  setup() {
    const canvasWidth = computed(() => DEFAULT_CELL_WIDTH * table.titles.length + DEFAULT_CELL_WIDTH)
    const spreadsheetRef = ref()
    const wrapperRef = ref()
    const tableWithCoords = ref(table)

    function createSpreadsheet() {
      if (spreadsheetRef.value.getContext) {
        const ctx = spreadsheetRef.value.getContext('2d')

        setCtxStyles(ctx)
        table.titles.forEach((t, id) => {
          const titleCell = new SpreadsheetCell(t.title, id, 0, setXCoord(id))
          titleCell.createCell(ctx)
        })

        table.children.forEach((v, vId) => {
          const lineId = vId + 1

          const nameCell = new SpreadsheetCell(v.name, lineId, lineId, X_OF_FIRST_CELL)
          nameCell.createCell(ctx)

          v.coords = {
            x: 0,
            y: setCoordY(lineId)
          }

          v.days.forEach((d, id) => {
            const cell = new SpreadsheetCell(d.data, id, lineId, setXCoord(id))
            cell.createCell(ctx)

            d.coords = {
              x: setXCoord(id),
              y: setCoordY(lineId)
            }
          })
        })
      } else {
        console.log('не поддерживается контекст')
      }
    }

    function setXCoord(id) {
      return DEFAULT_CELL_WIDTH + DEFAULT_CELL_WIDTH * id
    }

    function setCtxStyles(ctx) {
      ctx.strokeStyle = 'grey'
      ctx.lineWidth = 1
      ctx.font = '18px serif'
    }

    function handleCanvas(e) {
      if (!tableWithCoords.value) {
        return
      }

      const x = e.offsetX
      const y = e.offsetY

      let inputCellY
      let inputCellX
      let cache

      tableWithCoords.value.children.forEach((v) => {
        v.days.forEach((d) => {
          const isInsideX = x > d.coords.x && x < d.coords.x + DEFAULT_CELL_WIDTH
          const isInsideY = y > d.coords.y && y < d.coords.y + DEFAULT_CELL_HEIGHT

          if (isInsideX) {
            inputCellX = d.coords.x
          }

          if (isInsideY) {
            inputCellY = d.coords.y
          }

          if (isInsideY && isInsideX) {
            cache = d.data
          }
        })
      })

      if (inputCellX >= 0 && inputCellY >= 0) {
        removePrevInputCell()

        const inputCell = new InputCell(inputCellX, inputCellY, cache)

        wrapperRef.value.appendChild(inputCell.createInputCell())
      }
    }

    class InputCell {
      constructor(x, y, cache = '') {
        this.x = x
        this.y = y
        this.cache = cache
      }

      createInputCell() {
        const inputCell = document.createElement('div')
        const input = document.createElement('input')
        input.setAttribute('type', 'text')
        input.value = this.cache

        inputCell.classList.add('cell')
        inputCell.setAttribute('id', 'js-input-cell')
        inputCell.style.position = 'absolute'
        inputCell.style.width = DEFAULT_CELL_WIDTH + 'px'
        inputCell.style.height = DEFAULT_CELL_HEIGHT + 'px'
        inputCell.style.top = this.y + 'px'
        inputCell.style.left = this.x + 'px'

        inputCell.appendChild(input)
        return inputCell
      }
    }

    class SpreadsheetCell {
      constructor(data, id, lineId, coordX) {
        this.data = data
        this.id = id
        this.lineId = lineId
        this.coordX = coordX
      }

      coordY() {
        return setCoordY(this.lineId)
      }

      textCoordY() {
        return this.coordY() + DEFAULT_FONT_SIZE
      }

      textTextX() {
        return this.coordX + 10
      }

      createCell(ctx) {
        ctx.strokeRect(this.coordX, this.coordY(), DEFAULT_CELL_WIDTH, DEFAULT_CELL_HEIGHT)
        ctx.fillText(this.data, this.textTextX(), this.textCoordY())
      }
    }

    function removePrevInputCell() {
      const wrapper = document.querySelector('#js-main-wrapper')
      const inputCell = wrapper.querySelector('#js-input-cell')

      if (inputCell) {
        wrapper.removeChild(inputCell)
      }
    }

    function setCoordY(id) {
      return DEFAULT_CELL_HEIGHT * id
    }

    onMounted(() => {
      spreadsheetRef.value.width = '450'
      createSpreadsheet()
    })

    return { spreadsheetRef, handleCanvas, wrapperRef, canvasWidth }
  }
}
</script>

<style lang="css">
.wrapper {
  position: relative;
  /* border: 2px solid red; */
  height: 200px;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
}

.cell {
  outline: 2px solid green;
  position: absolute;
  overflow: hidden;
}

.cell input {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}
</style>
