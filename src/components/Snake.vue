<template>
  <div class="snake">
    <div class="top">
      <h1>贪吃蛇</h1>
      <h3>无尽模式</h3>
      <div class="show-action">
        <div class="show">
          <h4>当前分数: {{ mark }}</h4>
          <h4>最高分：{{ max_mark }}</h4>
        </div>
        <div class="action">
          <button class="restart"@click="restart">重新开始(R)</button>
          <button class="pause"@click="pause">暂停(SPACE)</button>
        </div>
      </div>
    </div>
    <div class="content">
      <table>
        <tr v-for="row in height">
          <td v-for="column in width":class="{ body: snake.body.indexOf(row+','+column) !== -1, food: row+','+column === food}"></td>
        </tr>
      </table>
      <div class="over-block" :class="{ over: status==='stoped' }">
        <h2>游戏结束</h2>
        <p>最后得分： {{ mark }}</p>
        <h3 v-if="mark >= max_mark">！！！最高分！！！</h3>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'snake',
  data () {
    return {
      mark: 0,
      max_mark: 1,
      height: 20,
      width: 32,
      snake: {
        body: ['2,5', '2,4', '2,3', '2,2', '2,1'],
        head: [2, 5],
        direction: {
          cur: 'right',
          next: 'right',
          canChange: true
        },
        speed: 300
      },
      food: '10,15',
      status: 'paused',
      animation: ''
    }
  },
  created () {
    this.showFood()
  },
  mounted () {
    this.start()
    document.addEventListener('keydown', this.eventListener)
  },
  methods: {
    restart () {
      this.snake = {
        body: ['2,5', '2,4', '2,3', '2,2', '2,1'],
        head: [2, 5],
        direction: {
          cur: 'right',
          next: 'right',
          canChange: true
        },
        speed: 300
      }
      this.showFood()
      clearTimeout(this.animation)
      this.status = 'paused'
      this.mark = 0
      this.start()
    },
    start () {
      this.animation = setTimeout(this.start, this.snake.speed)
      this.status = 'running'
      this.move()
    },
    pause () {
      if (this.status === 'running') {
        clearTimeout(this.animation)
        this.status = 'paused'
      } else {
        this.start()
      }
    },
    changeDirection (event) {
      var direction = ''
      switch (event.code) {
        case 'ArrowUp': direction = 'up'
          break
        case 'ArrowDown': direction = 'down'
          break
        case 'ArrowLeft': direction = 'left'
          break
        case 'ArrowRight': direction = 'right'
          break
      }
      if (((this.snake.direction.cur === 'right' || this.snake.direction.cur === 'left') &&
      (direction === 'up' || direction === 'down')) ||
      ((this.snake.direction.cur === 'up' || this.snake.direction.cur === 'down') &&
      (direction === 'left' || direction === 'right'))) {
        if (this.snake.direction.canChange) {
          this.snake.direction.cur = direction
          this.snake.direction.next = direction
          this.snake.direction.canChange = false
        } else {
          this.snake.direction.next = direction
        }
      }
    },
    eventListener (event) {
      event.preventDefault()
      this.changeDirection(event)
      if (event.code === 'Space') {
        this.pause()
      } else if (event.code === 'KeyR') {
        this.restart()
      }
    },
    move () {
      var head = this.snake.head
      var direction = this.snake.direction.cur
      head[0] = direction === 'up' ? head[0] - 1 : direction === 'down' ? head[0] + 1 : head[0]
      head[1] = direction === 'left' ? head[1] - 1 : direction === 'right' ? head[1] + 1 : head[1]

      if (this.isOver()) {
        clearTimeout(this.animation)
        return
      }
      this.snake.body.unshift(head[0] + ',' + head[1])
      if (head[0] + ',' + head[1] !== this.food) {
        this.snake.body.pop()
      } else {
        this.showFood()
        this.mark += 10
      }
      if (!this.snake.direction.canChange) {
        this.snake.direction.canChange = true
        this.snake.direction.cur = this.snake.direction.next
      }
    },
    showFood () {
      var row = Math.ceil(Math.random() * this.height)
      var col = Math.ceil(Math.random() * this.width)
      while (this.snake.body.indexOf(row + ',' + col) !== -1) {
        row = Math.ceil(Math.random() * this.height)
        col = Math.ceil(Math.random() * this.width)
      }
      this.food = row + ',' + col
    },
    isOver () {
      var head = this.snake.head
      if (head[0] < 1 || head[1] < 1 || head[0] > this.height || head[1] > this.width ||
      this.snake.body.indexOf(head[0] + ',' + head[1]) !== -1) {
        this.status = 'stoped'
        return true
      }
      return false
    }
  }
}
</script>

<style lang="sass">
.snake
  margin: auto;

  h1,h3
    text-align: center;
    margin-bottom: 3px;

  h3
    color: grey;
    margin: 0;
  .show-action 
    display: flex;
    
  .show-action>.show
    flex: 1.5;
  
  .show-action>.show
    flex: 1;

  .action
    display: flex;
    flex-direction: column;
    justify-content: space-around;

.content
  border: 2px solid black;
  position: relative

  .over-block
    position: absolute;
    top: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    color: white
    text-align: center;
    flex-direction: column;
    justify-content: center;
    display: none;
  h3
    color: white;
  .over
    display: flex

td
  height: 15px;
  width: 15px;

td.body
  background: #595959;

td.food
  background: #595959;
  border-radius: 50%;
</style>
