<template>
  <v-container class="grey lighten-5">
    <v-row no-gutters>
      <v-col cols="12" sm="4"></v-col>
      <v-col cols="12" sm="4">
        <!-- Card Start -->
        <v-card class outlined tile>
          <ul @click="updateMessage" ref="messageBox" id="messageBox">
            <li v-for="message in showData" :key="message.id">
              <v-chip v-if="message.data" color="black">
                <span class="custom-text">{{ message.data }}</span>
              </v-chip>
              <div style="margin-top: .5em" v-if="isAttachment(message)">
                <v-img
                  src="https://picsum.photos/id/11/500/300"
                  lazy-src="https://picsum.photos/id/11/10/6"
                  aspect-ratio="1"
                  class="grey lighten-2 z-2"
                  max-width="500"
                  max-height="300"
                ></v-img>
              </div>
              <div v-if="message.type === 'question'">
                <v-btn
                  small
                  v-for="(question, index) in message.questions"
                  :key="index"
                  @click="questionClick(question)"
                  class="my-btn"
                  :disabled="!question.active"
                  color="primary"
                >{{question.message}}</v-btn>
              </div>
            </li>
          </ul>
        </v-card>
        <!-- Card End -->
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import sampleData from "../sample.json";
export default {
  data() {
    return {
      sampleData,
      isQuestionClick: false,
      showData: [sampleData.messages[0]],
      currentId: sampleData.messages[0].id
    };
  },
  methods: {
    updateMessage() {
      const {
        getCurrentMessage,
        isFinished,
        getNextMessage,
        isQuestionType,
        isQuestionClick
      } = this;
      const currentMessage = getCurrentMessage();
      if (isFinished(currentMessage)) {
        /**
        |--------------------------------------------------
        | Finished Coming Component
        |--------------------------------------------------
        */
        return true;
      }

      if (isQuestionType() || isQuestionClick) {
        if (isQuestionClick) this.isQuestionClick = false;
        return true;
      }

      let nextMessage = getNextMessage();
      if (nextMessage.type === "question") {
        nextMessage.questions = nextMessage.questions.map(ques => {
          return { ...ques, active: true };
        });
      }
      this.currentId = nextMessage.id;
      this.showData = [...this.showData, nextMessage];
      this.scrollDown();
    },
    isFinished(message) {
      return message.finish === true;
    },
    getCurrentMessage() {
      const { sampleData, currentId } = this;
      return sampleData.messages.find(mes => mes.id === currentId);
    },
    getNextMessage() {
      const { sampleData, getCurrentMessage } = this;
      const currentMessage = getCurrentMessage();
      return sampleData.messages.find(mes => mes.id === currentMessage.to);
    },
    questionClick(question) {
      const { sampleData, setQuestionFalse } = this;
      const nextMessage = sampleData.messages.find(
        mes => mes.id === question.to
      );
      setQuestionFalse();
      this.currentId = nextMessage.id;
      this.showData = [...this.showData, nextMessage];
      this.isQuestionClick = true;
      this.scrollDown();
    },
    isQuestionType() {
      return this.getCurrentMessage().type === "question";
    },
    isAttachment(message) {
      return message.attachment;
    },
    setQuestionFalse() {
      const { getCurrentMessage } = this;
      const currentMessage = getCurrentMessage();
      this.showData = this.showData.map(data => {
        let returnData = data;
        if (returnData.id === currentMessage.id) {
          returnData.questions = returnData.questions.map(ques => {
            return { ...ques, active: false };
          });
        }
        return returnData;
      });
    },
    scrollDown() {
      setTimeout(() => {
        this.$refs.messageBox.scrollTop = this.$refs.messageBox.scrollHeight;
      }, 4);
    }
  }
};
</script>

<style scoped>
.custom-text {
  color: white;
}
.my-btn {
  /* border-radius: 1rem; */
  margin: 0.3em 0;
  padding: 0 5px !important;
  margin-left: 1px;
  margin-right: 1px;
  margin-top: 0.2em;
  margin-bottom: 0.2em;
  z-index: 2;
}
.z-2 {
  width: 80%;
  border-radius: 5px;
  z-index: 2;
}

ul {
  list-style-type: none;
  margin: 0;
  padding: 5px !important;
  min-height: 80vh;
  max-height: 80vh;
  overflow-y: scroll;
  cursor: pointer;
}

ul li {
  margin-bottom: 1em;
}

ul li .v-chip {
  transform-origin: 50% 50%;
  animation: message 0.3s ease-in-out forwards;
}

ul::after {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 100%;
  background-image: linear-gradient(
    rgb(0, 95, 255) 0%,
    rgb(146, 0, 255) 50%,
    rgb(255, 46, 25) 100%
  );
  mix-blend-mode: screen;
  pointer-events: none;
}

ul::-webkit-scrollbar {
  width: 0.4em;
}

ul::-webkit-scrollbar-track {
  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  border-radius: 4px;
}

ul::-webkit-scrollbar-thumb {
  background-color: darkgrey;
  outline: 1px solid slategrey;
  border-radius: 4px;
}

@keyframes message {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
</style>
