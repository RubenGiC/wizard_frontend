<template>
  <div id="app">
    <div class="chat-container" ref="chatContainer">
      <div id="total" class=" container mb-3 sticky-top p-2">
        <div class="row text-center">
          <div class="col-3"><strong>Gryffindor</strong></div>
          <div class="col-3"><strong>Ravenclaw</strong></div>
          <div class="col-3"><strong>Hufflepuff</strong></div>
          <div class="col-3"><strong>Slytherin</strong></div>
        </div>
        <div class="row text-center">
          <div class="col">{{ totalScores.g/4 }}%</div>
          <div class="col">{{ totalScores.r/4 }}%</div>
          <div class="col">{{ totalScores.h/4 }}%</div>
          <div class="col">{{ totalScores.s/4 }}%</div>
        </div>
        <div class="row text-center">
          <div class="col">{{ totalScores.g }}</div>
          <div class="col">{{ totalScores.r }}</div>
          <div class="col">{{ totalScores.h }}</div>
          <div class="col">{{ totalScores.s }}</div>
        </div>
      </div>
      <div
        v-for="(msg, idx) in messages"
        :key="idx"
        :class="['chat-bubble', msg.from]"
        :style="{ maxWidth: '80%' }"
      >
        <template v-if="msg.type === 'question'">
          <div class="question-text">{{ msg.text }}</div>
          <div class="options" v-if="msg.options">
            <button
              v-for="opt in msg.options"
              :key="opt.scores"
              class="option-btn"
              @click="answer(idx, opt.title)"
            >
              {{ opt.title }}
            </button>
          </div>
        </template>
        <template v-else>
          <div class="answer-text">{{ msg.text }}</div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import questionsData from './sorting_hat.json';

export default {
  name: 'App',
  data() {
    return {
      questions: [],
      messages: [],
      currentQuestion: 0,
      totalScores: { g: 0, r: 0, h: 0, s: 0 }
    };
  },
  created() {
    this.questions = questionsData;
    if (this.questions.length > 0) {
      this.messages.push({
        type: 'question',
        from: 'bot',
        text: this.questions[0].title,
        options: this.questions[0].answers
      });
    }
  },
  methods: {
    answer(idx, opt) {
      // Buscar la opción seleccionada en la pregunta actual
      const currentQ = this.questions[this.currentQuestion];
      let selectedOption = null;
      console.log('opt:', opt);
      console.log('currentQ:', currentQ);
      if (currentQ && currentQ.answers) {
        console.log('options:', currentQ.answers);
        selectedOption = currentQ.answers.find(o => (o.title || o) === opt);
      }
      console.log('selectedOption.scores:', selectedOption);
      // Si la opción tiene scores, sumarlos
      if (selectedOption && selectedOption.scores) {
        for (const key in selectedOption.scores) {
          if (Object.prototype.hasOwnProperty.call(this.totalScores, key)) {
            this.totalScores[key] += selectedOption.scores[key];
          }
        }
      }
      // Añade la respuesta del usuario
      this.messages.push({
        type: 'answer',
        from: 'user',
        text: opt
      });

      // Siguiente pregunta
      this.currentQuestion++;
      if (this.questions[this.currentQuestion]) {
        setTimeout(() => {
          this.messages.push({
            type: 'question',
            from: 'bot',
            text: this.questions[this.currentQuestion].title,
            options: this.questions[this.currentQuestion].answers
          });
        }, 400);
      }
      this.$nextTick(this.scrollToBottomSmooth);
    },
    scrollToBottomSmooth() {
      // Desplazamiento suave al último mensaje
      const el = this.$refs.chatContainer;
      if (el) {
        el.scrollTo({
          top: el.scrollHeight,
          behavior: 'smooth'
        });
      }
    }
  },
  updated() {
    this.scrollToBottomSmooth();
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  background: #faf4e2;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}
#title_app{
  background-color: #faf4e2;
}

#total{
  background: #faf4e2;
  border: 1px solid #ffd64f;
  border-radius: 5px;
}

.chat-container {
  width: 100%;
  max-width: 420px;
  height: 70vh;
  background: #fff;
  border-radius: 18px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.07);
  padding: 18px 8px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
  scroll-behavior: smooth;
}

.chat-bubble {
  padding: 14px 18px;
  border-radius: 18px;
  margin-bottom: 2px;
  word-break: break-word;
  animation: fadeInUp 0.35s;
  transition: background 0.2s;
}

.bot {
  background: #faf4e2;
  align-self: flex-start;
}

.user {
  background: #ffd64f;
  color: #fff;
  align-self: flex-end;
}

.options {
  margin-top: 10px;
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.option-btn {
  background: #fff;
  border: 1px solid #ffd64f;
  color: #f5c015;
  border-radius: 12px;
  padding: 7px 18px;
  cursor: pointer;
  font-size: 1em;
  transition: background 0.2s, color 0.2s;
}
.option-btn:hover, .option-btn:focus {
  background: #ffd64f;
  color: #fff;
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(24px);}
  to { opacity: 1; transform: translateY(0);}
}
</style>