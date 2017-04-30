<template>
  <div class="snake">
    <div class="top">
      <h1>贪吃蛇</h1>
      <h3><select name="type" id="type"@change="typeChanged":disabled="status==='running'?true:false">
        <option value=-1 selected>无尽模式</option>
        <option value=0>闯关模式</option>
      </select></h3>
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
      <h3 v-if="type !== -1">第 {{ type+1 }} 关</h3>
    </div>
    <div class="content">
      <table cellspacing="0">
        <tr v-for="row in height">
          <td v-for="col in width":class="{ body: snake.body.indexOf(row+','+col) !== -1, food: row+','+col === food, wall: type !== -1 && walls[type].indexOf(row+','+col) !== -1}"></td>
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
      type: -1,
      snake: {
        body: ['10,16', '10,15', '10,14', '10,13', '10,12'],
        head: [10, 16],
        direction: {
          cur: 'right',
          next: 'right',
          canChange: true
        },
        speed: 300,
        throughBorder: true,
        eaten: 0
      },
      food: '10,15',
      walls: [
        ['2,2', '2,3', '2,4', '2,5', '2,6', '2,7', '3,2', '4,2',
          '3,31', '4,31', '2,31', '2,30', '2,29', '2,28', '2,27', '2,26',
          '19,2', '19,3', '19,4', '19,5', '19,6', '19,7', '18,2', '17,2',
          '18,31', '17,31', '19,31', '19,30', '19,29', '19,28', '19,27', '19,26'],
        [
          '7,10', '7,11', '7,12', '7,13', '7,14', '7,15', '7,16', '7,17', '7,18', '7,19', '7,20', '7,21', '7,22',
          '13,10', '13,11', '13,12', '13,13', '13,14', '13,15', '13,16', '13,17', '13,18', '13,19', '13,20', '13,21', '13,22'
        ]
      ],
      status: 'paused',
      animation: ''
    }
  },
  created () {
    this.showFood()
  },
  mounted () {
    this.start()
    document.addEventListener('keydown', this.keyListener)
  },
  methods: {
    restart () {
      var _this = this
      this.snake = {
        body: ['10,16', '10,15', '10,14', '10,13', '10,12'],
        head: [10, 16],
        direction: {
          cur: 'right',
          next: 'right',
          canChange: true
        },
        speed: 300,
        eaten: 0,
        throughBorder: _this.snake.throughBorder
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
      } else if (this.status === 'paused') {
        this.start()
      }
    },
    typeChanged (event) {
      console.log(event.target.value)
      this.type = Number(event.target.value) > 0 ? 0 : Number(event.target.value)
      this.restart()
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
    keyListener (event) {
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

      if (this.snake.throughBorder) {
        head[0] = head[0] < 1 ? this.height : head[0] > this.height ? head[0] % this.height : head[0]
        head[1] = head[1] < 1 ? this.width : head[1] > this.width ? head[1] % this.width : head[1]
      }
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
        this.snake.eaten ++
        if ((this.type === 0) && this.snake.eaten >= 10 + 10 * this.type) {
          this.nextLevel(this.mark)
        }
      }
      if (!this.snake.direction.canChange) {
        this.snake.direction.canChange = true
        this.snake.direction.cur = this.snake.direction.next
      }
    },
    nextLevel (mark) {
      this.type ++
      this.restart()
      this.mark += mark
    },
    showFood () {
      var row = Math.ceil(Math.random() * this.height)
      var col = Math.ceil(Math.random() * this.width)
      while (this.snake.body.indexOf(row + ',' + col) !== -1 &&
      this.walls[this.type].indexOf(row + ',' + col) !== -1) {
        row = Math.ceil(Math.random() * this.height)
        col = Math.ceil(Math.random() * this.width)
      }
      this.food = row + ',' + col
    },
    isOver () {
      var head = this.snake.head
      if (this.snake.body.indexOf(head[0] + ',' + head[1]) !== -1) {
        this.status = 'stoped'
        return true
      }
      if (this.type !== -1 && this.walls[this.type].indexOf(head[0] + ',' + head[1]) !== -1) {
        this.status = 'stoped'
        return true
      }
      if ((!this.snake.throughBorder) &&
      (head[0] < 1 || head[1] < 1 || head[0] > this.height ||
      head[1] > this.width)) {
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
  background-image: url(../assets/body.png);
  background-size: 100%;

td.food
  background: #595959;
  border-radius: 50%;
td.wall
  background: black;
</style>
