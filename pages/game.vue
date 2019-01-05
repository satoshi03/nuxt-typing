<template>
  <section class="container">
    <div class="question-panel">
      <span>
        {{ getMessage() }}
      </span>
    </div>
    <div class="input-panel">
      <input @keyup.escape="reset()" @keyup.space="beforeStart()" v-model="answer"></input>
    </div>
    <div class="keyboard">
      <div
        v-for="(rowkeys, index) in keys"
        class="row"
        >
        <div
          v-for="item in keys[index]"
          :class="getKeyClass(item)"
          >
          {{ item.repl }}
        </div>
      </div>
    </div>
    </div>
  </section>
</template>

<script>
import lodash from 'lodash'

export default {
  data() {
    const words = ["print", "for", "if", "dict", "list", "class", "def", "except", "try", "error",
                   "int", "bool", "float", "str", "import", "from", "math", "set", "random"]
    const questionNum = 10
    return {
      allWords: words,
      words: lodash.shuffle(words),
      status: "ready",
      questionNum : questionNum,
      question: "",
      answeredNum: 0,
      wordCount: 0,
      answer: "",
      startTime: 3,
      timer: undefined,
      timerStepMs: 100,
      time: 0,
      isCorrectLastKey: true,
      lastCorrectKeyIndex: 0,
      wrongKeys: [],
      keys : [
        [
          { 'repl': '1', 'def': '1', 'cap': '!' },
          { 'repl': '2', 'def': '2', 'cap': '"' },
          { 'repl': '3', 'def': '3', 'cap': '#' },
          { 'repl': '4', 'def': '4', 'cap': '$' },
          { 'repl': '5', 'def': '5', 'cap': '%' },
          { 'repl': '6', 'def': '6', 'cap': '&' },
          { 'repl': '7', 'def': '7', 'cap': '\'' },
          { 'repl': '8', 'def': '8', 'cap': '(' },
          { 'repl': '9', 'def': '9', 'cap': ')' },
          { 'repl': '0', 'def': '0', 'cap': '0' },
          { 'repl': '-', 'def': '-', 'cap': '=' },
          { 'repl': '^', 'def': '^', 'cap': '~' },
          { 'repl': '\\', 'def': '\\', 'cap': '|' },
          { 'repl': 'DEL', 'def': '[DEL]', 'cap': '[DEL]', 'class': 'key' },
        ],
        [
          { 'repl': 'Q', 'def': 'q', 'cap': 'Q' },
          { 'repl': 'W', 'def': 'w', 'cap': 'W' },
          { 'repl': 'E', 'def': 'e', 'cap': 'E' },
          { 'repl': 'R', 'def': 'r', 'cap': 'R' },
          { 'repl': 'T', 'def': 't', 'cap': 'T' },
          { 'repl': 'Y', 'def': 'y', 'cap': 'Y' },
          { 'repl': 'U', 'def': 'u', 'cap': 'U' },
          { 'repl': 'I', 'def': 'i', 'cap': 'I' },
          { 'repl': 'O', 'def': 'o', 'cap': 'O' },
          { 'repl': 'P', 'def': 'p', 'cap': 'P' },
          { 'repl': '@', 'def': '@', 'cap': '`' },
          { 'repl': '[', 'def': '[', 'cap': '{' },
          { 'repl': 'Enter', 'def': '\n', 'cap': '\n', 'class': 'key--wide'},
        ],
        [
          { 'repl': 'A', 'def': 'a', 'cap': 'A' },
          { 'repl': 'S', 'def': 's', 'cap': 'S' },
          { 'repl': 'D', 'def': 'd', 'cap': 'D' },
          { 'repl': 'F', 'def': 'f', 'cap': 'F' },
          { 'repl': 'G', 'def': 'g', 'cap': 'G' },
          { 'repl': 'H', 'def': 'h', 'cap': 'H' },
          { 'repl': 'J', 'def': 'j', 'cap': 'J' },
          { 'repl': 'K', 'def': 'k', 'cap': 'K' },
          { 'repl': 'L', 'def': 'l', 'cap': 'L' },
          { 'repl': ';', 'def': ';', 'cap': '+' },
          { 'repl': ':', 'def': ':', 'cap': '*' },
          { 'repl': ']', 'def': ']', 'cap': '}' },
        ],
        [
          { 'repl': 'Shift', 'def': '[SHIFT]', 'cap': '[SHIFT]', 'class': 'key--wide'},
          { 'repl': 'Z', 'def': 'z', 'cap': 'Z' },
          { 'repl': 'X', 'def': 'x', 'cap': 'X' },
          { 'repl': 'C', 'def': 'c', 'cap': 'C' },
          { 'repl': 'V', 'def': 'v', 'cap': 'V' },
          { 'repl': 'B', 'def': 'b', 'cap': 'B' },
          { 'repl': 'N', 'def': 'n', 'cap': 'N' },
          { 'repl': 'M', 'def': 'm', 'cap': 'M' },
          { 'repl': ',', 'def': ',', 'cap': '<' },
          { 'repl': '.', 'def': '.', 'cap': '>' },
          { 'repl': '/', 'def': '/', 'cap': '?' },
          { 'repl': '_', 'def': '_', 'cap': '_' },
          { 'repl': 'Shift', 'def': '[SHIFT]', 'cap': '[SHIFT]', 'class': 'key--wide'},
        ],
        [
          { 'repl': '', 'def': ' ', 'cap': ' ', 'class': 'key--space'},
        ],
      ],
    }
  },
  computed: {
    getTime() {
      return Number((this.time).toFixed(1))
    },
  },
  methods: {
    beforeStart() {
      if (this.status == "starting" || this.status == "started") {
        return
      }

      if (this.status == "finished") {
        this.reset()
      }

      // Count down start
      this.status = "starting"
      this.answer = ""
      this.timer = setInterval(this.startCount, 1000)
    },
    startCount() {
      this.startTime -= 1
      if (this.startTime <= 0) {
        clearInterval(this.timer)
        this.start()
      }
    },
    start() {
      this.status = "started"
      this.timer = setInterval(this.timerCount, this.timerStepMs)
      this.answer = ""
      this.question = this.words.pop()
    },
    timerCount() {
      this.time += this.timerStepMs / 1000
    },
    getMessage() {
      // return instuction when status is ready
      if (this.status == "ready") {
        return "下の入力フォームでスペースキーを押すと始まります"
      }

      // return count down time when status is starting
      if (this.status == "starting") {
        return this.startTime
      }

      // return question when status is started
      if (this.status == "started") {
        if (this.isWrongAnwser()) {
          if (this.isCorrectLastKey) {
            this.wrongKeys.push(this.answer[this.answer.length - 1])
          }
          this.isCorrectLastKey = false
        } else {
          this.isCorrectLastKey = true
          this.lastCorrectKeyIndex = this.answer.length
        }
        if (this.isCompleteAnswer()) {
          this.answeredNum += 1
          this.wordCount += this.question.length
          this.question = this.words.pop()
          this.lastCorrectKeyIndex = 0
          this.answer = ""
        }
        if (this.isFinished()) {
          this.status = "finished"
        }
        return this.question
      }

      // return results when status is finished
      if (this.status = "finished") {
        clearInterval(this.timer)
        return this.getResultMessage()
      }
    },
    getResultMessage() {
      var speed = Number(this.wordCount / this.time * 60).toFixed(1)
      var correctParcent = Number(this.wordCount / (this.wordCount + this.wrongKeys.length) * 100).toFixed(1)
      return "タイピング速度:" + speed + "WPM, 正答率:" + correctParcent + "%"
    },
    isWrongAnwser() {
      return (this.answer != this.question.slice(0, this.answer.length))
    },
    getNextKey() {
      if (this.status == "ready" || this.status == "finished") {
        return " "
      }
      return this.question[this.lastCorrectKeyIndex]
    },
    getWrongKey() {
      if (this.isCorrectLastKey) {
        return ""
      } else {
        return this.wrongKeys[this.wrongKeys.length - 1]
      }
    },
    isFinished() {
      return this.answeredNum >= this.questionNum
    },
    isCompleteAnswer() {
      return this.answer == this.question
    },
    getKeyClass(item) {
      var classes = []
      if (item.class) {
        classes.push(item.class)
      } else {
        classes.push("key")
      }
      var lastStr = this.getNextKey()
      if (item.def == lastStr || item.cap == lastStr) {
        classes.push("key--push")
      }
      var wrongKey = this.getWrongKey()
      if (wrongKey !== "" && (item.def == wrongKey || item.cap == wrongKey)) {
        classes.push("key--wrong")
      }
      return classes
    },
    reset() {
      this.question = ""
      this.answer = ""
      this.words = lodash.shuffle(this.allWords)
      clearInterval(this.timer)
      this.startTime = 3
      this.time = 0
      this.status = "ready"
      this.answeredNum = 0
      this.wordCount = 0
      this.isCorrectLastKey = true
      this.wrongKeys = []
    },
  }
}
</script>

<style>
.container {
  display: flex;
  flex-direction:column;
  justify-content: center;
  align-items: center;
}

.question-panel {
  display:flex;
  height: 150px;
  width: 80%;
  margin: 5px;
  margin-top: 20px;
  flex-direction:column;
  justify-content: center;
  align-items: center;
  border: 1px solid #35495e;
  border-radius: 10px;
}

span {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  font-size: 30px;
  color: #35495e;
}

.input-panel {
  display:flex;
  align-items:center;
  justify-content: center;
  height: 100px;
  width: 80%;
}

input {
  width: 80%;
  height: 40px;
  font-size: 30px;
  border-radius: 4px;
  -webkit-border-radius: 3px;
  -moz-border-radius: 3px;
  color: #35495e;
}

.keyboard {
  display:flex;
  flex-direction:column;
  align-items:center; /* Y-axes in this case */
}

.row {
  display:flex;
  align-content:space-around;
}

.key {
  padding:10px;
  height:60px;
  width:60px;
  border: 1px solid black;
  border-right: 2px solid black;
  border-bottom: 2px solid black;
  border-radius: 4px;
  margin: 2px;
}

.key--wide {
  padding:10px;
  height:60px;
  width:100px;
  border: 1px solid black;
  border-right: 2px solid black;
  border-bottom: 2px solid black;
  border-radius: 4px;
  margin: 2px;
}

.key--push {
  border: 1px solid black;
  background-color: #64B5F6;
}

.key--wrong {
  padding:10px;
  height:60px;
  width:60px;
  border: 1px solid black;
  border-radius: 4px;
  margin: 2px;
  background-color: #E53935;
}

.key--space {
  padding:10px;
  height:60px;
  width:400px;
  border: 1px solid black;
  border-right: 2px solid black;
  border-bottom: 2px solid black;
  border-radius: 4px;
  margin: 2px;
}
</style>
