<template>
  <div v-if="currentScreen === 'main'" id="mainScreen">
    <h1 id="gameTitle">Memory Dungeon</h1>
    <button id="btStart" @click="startGame">CLICK TO START</button>
  </div>
  <div v-if="currentScreen === 'game'" id="gameScreen">
    <div id="player">
      <div class="field">
        <img
          class="icon"
          v-for="index in hearts"
          :key="index"
          :src="getHeartImage(index)"
        />
      </div>
      <div class="field">
        <img class="icon" src="/dungeon.png" />
        <p>{{ currentRoom }}</p>
      </div>
      <div class="field">
        <img class="icon" src="/sword.png" />
        <p>{{ attack }}</p>
      </div>

      <div class="field">
        <img class="icon" src="/coins.png" />
        <p>{{ gold }}</p>
      </div>
    </div>

    <button
      id="btSlaySword"
      v-if="attack > 0 && flippedCards === 0"
      @click="slayMonster"
    >
      <img class="icon" src="/sword.png" />
      <span>Use</span>
    </button>
    <div id="notification">
      {{ message }}
    </div>
    <div id="board">
      <div
        class="card"
        v-for="card in cards"
        @click="flip(card)"
        :class="{ flipped: card.flipped }"
      >
        <div v-if="card.flipped === false" class="face">
          <img src="/dungeon.png" alt="" />
        </div>
        <div v-else class="back">
          <img
            :src="'/' + card.name + '.png'"
            :style="`border:2px solid var(--${card.color})`"
          />
        </div>
      </div>
    </div>
  </div>

  <!-- modal confirm -->
  <ModalConfirm :isVisible="isModalConfirmVisible" @confirm="handleConfirm">
    <template v-slot:content>
      {{ messageConfirm }}
    </template>
  </ModalConfirm>
</template>

<script>
import ModalConfirm from "./components/ModalConfirm.vue";

export default {
  name: "App",
  components: {
    ModalConfirm,
  },
  data() {
    return {
      currentScreen: "main",
      flippedCards: 0,
      cardsToCompare: [],
      gold: 0,
      hearts: 3,
      filledHearts: 3,
      attack: 0,
      currentRoom: 1,
      message: "",
      // modal confirm
      isModalConfirmVisible: false,
      messageConfirm: "",
      isValidateConfirm: false,
      cards: [],
    };
  },
  mounted() {
    // this.startGame();
  },
  methods: {
    showScreen(screen) {
      this.currentScreen = screen;
    },
    showModalConfirm(message) {
      this.messageConfirm = message;
      this.isModalConfirmVisible = true;
      return new Promise((resolve) => {
        this.isValidateConfirm = resolve;
      });
    },
    //------------------------------------------------------------------------------------------
    handleConfirm() {
      this.messageConfirm = "";
      this.isModalConfirmVisible = false;
      this.isValidateConfirm(true);
    },

    shuffleArray(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    },
    startGame() {
      let msg = `Bienvenue dans Memory Dungeon.
      Parcourez le donjon, trouvez des paires,  
ramassez les épées afin  
de vaincre les monstres sans difficultés,  
sinon il vous en coûtera.  

Serez-vous assez courageux  
pour obtenir les 10 pièces d'or ?`;
      this.showModalConfirm(msg).then((result) => {
        if (result) {
          this.resetBoard();
          this.resetPlayer();
          this.generateRandomBoard();
          this.shuffleArray(this.cards);
          this.showScreen("game");
        }
      });
    },
    cardExists(card, array) {
      return array.some((existingCard) => existingCard.name === card.name);
    },
    generateRandomBoard() {
      this.cards = [];
      let items = [
        {
          name: "potion",
          color: "red",
        },
        {
          name: "monster",
          color: "violet",
        },
        {
          name: "coins",
          color: "yellow",
        },
        {
          name: "sword",
          color: "blue",
        },
        {
          name: "shield",
          color: "brown",
        },
        {
          name: "beer",
          color: "yellow",
        },
      ];
      let names = ["potion", "monster", "coins", "sword", "shield", "beer"];
      let selectedCards = [];

      while (selectedCards.length < 4) {
        let index = Math.floor(Math.random() * items.length);

        let randomCard = {
          name: items[index].name,
          flipped: false,
          img: names[index],
          color: items[index].color,
        };

        if (!this.cardExists(randomCard, selectedCards)) {
          selectedCards.push(randomCard);
        }
      }
      for (let card of selectedCards) {
        let cardCopy = { ...card };
        this.cards.push(card, cardCopy);
      }
      let ladder = {
        name: "ladder",
        flipped: false,
        img: "ladder",
        color: "grey",
      };
      this.cards.push(ladder);
    },
    resetBoard() {
      this.cardsToCompare = [];
      this.flippedCards = 0;
      this.flippedBackCards();
    },
    resetPlayer() {
      this.attack = 0;
      this.currentRoom = 1;
      this.gold = 0;
      this.filledHearts = 3;
    },
    gameOver() {
      this.showModalConfirm("Game over!").then((result) => {
        if (result) {
          this.resetBoard();
          this.resetPlayer();
          this.generateRandomBoard();
          this.shuffleArray(this.cards);
          this.showScreen("main");
        }
      });
    },
    winGame() {
      this.showModalConfirm("You have won!").then((result) => {
        if (result) {
          this.resetBoard();
          this.resetPlayer();
          this.generateRandomBoard();
          this.shuffleArray(this.cards);
          this.showScreen("main");
        }
      });
    },
    flip(card) {
      if (card.flipped) {
        return;
      }
      this.flippedCards++;
      if (this.flippedCards > 2) {
        return;
      }

      card.flipped = true;
      if (card.name === "ladder" && this.checkIfAllCardsFlipped()) {
        setTimeout(() => {
          this.currentRoom++;
          this.resetBoard();
          this.generateRandomBoard();
          this.shuffleArray(this.cards);
          this.showMessage("Next room");
        }, 2000);
        return;
      }

      this.cardsToCompare.push(card);
      this.compareCard();

      if (this.filledHearts === 0) {
        this.gameOver();
        return;
      }
      if (this.gold === 10) {
        this.winGame();
        return;
      }
    },
    flippedBackCards() {
      for (let i = 0; i < this.cards.length; i++) {
        this.cards[i].flipped = false;
      }
    },
    checkIfAllCardsFlipped() {
      return this.cards.every((card) => card.flipped);
    },

    compareCard() {
      if (this.cardsToCompare.length !== 2) {
        return;
      }
      let card1 = this.cardsToCompare[0];
      let card2 = this.cardsToCompare[1];

      if (card1.name !== card2.name) {
        // console.log("not equal");
        setTimeout(() => {
          card1.flipped = false;
          card2.flipped = false;
          this.cardsToCompare = [];
          this.flippedCards = 0;
        }, 1000);
        return;
      }

      if (card1.name === "coins") {
        this.increaseGold();
      }

      if (card1.name === "sword") {
        this.increaseAttack();
      }
      if (card1.name === "monster") {
        this.decreaseHearts();
      }
      if (card1.name === "potion") {
        this.increaseHearts();
      }
      this.cardsToCompare = [];
      this.flippedCards = 0;
    },
    increaseGold() {
      this.gold++;
      this.showMessage(`You have found gold!`);
    },
    increaseAttack() {
      this.attack++;
      if (this.attack >= 1) {
        this.attack = 1;
      }
      this.showMessage(`You get a sword!`);
    },
    slayMonster() {
      for (let i = 0; i < this.cards.length; i++) {
        if (this.cards[i].name === "monster") {
          this.cards[i].flipped = true;
        }
      }
      this.attack = 0;
    },
    showMessage(text) {
      this.message = text;
      setTimeout(() => {
        this.message = "";
      }, 2000);
    },

    increaseHearts() {
      this.filledHearts++;
      if (this.filledHearts >= 3) {
        this.filledHearts = 3;
      }
      this.showMessage(`You have earned a health point!`);
    },
    decreaseHearts() {
      this.filledHearts--;
      if (this.filledHearts === 0) {
        this.filledHearts = 0;
      }
      this.showMessage("You have lost a health point!");
    },

    getHeartImage(index) {
      return index <= this.filledHearts ? "/heart-red.png" : "/heart-grey.png";
    },
  },
};
</script>
<style>
#mainScreen {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
  align-items: center;
  gap: 1rem;
}
#gameTitle {
  font-size: 48px;
}
#btStart {
  border: none;
  background: none;
  color: var(--white);
  padding: 0;
  font-size: 24px;
  animation: blink 2s ease-in-out infinite;
}
@keyframes blink {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
#gameScreen {
  display: flex;
  justify-content: center;
  flex-direction: column;
  height: 100vh;
}
p {
  margin: auto 0;
  font-size: 2.5rem;
  width: 50px;
}
#board {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  height: 300px;
  max-width: 300px;
  gap: 0.5rem;
  margin: auto;
}
.card {
  height: 100px;
  width: 100px;
  background: var(--background);
  transform-style: preserve-3d;
}

@keyframes flip {
  from {
    -webkit-transform: rotateY(0deg);
    transform: rotateY(0deg);
  }
  to {
    -webkit-transform: rotateY(180deg);
    transform: rotateY(180deg);
  }
}
.flipped {
  animation: flip 0.5s;
  transform: rotateY(180deg);
}
.face {
  background: var(--background);
  height: 100%;
  backface-visibility: hidden;
}

.back {
  background: var(--background);
  height: 100%;
  backface-visibility: hidden;
  transform: rotateY(180deg);
}

.icon {
  width: 34px;
  height: 34px;
}
.field {
  display: flex;
  text-align: center;
  align-items: center;
}
#player {
  background: var(--background);
  border: 2px solid var(--grey);
  display: flex;
  align-items: center;
  gap: 0.1rem;
}
#btSlaySword {
  display: flex;
  text-align: center;
  align-items: center;
  position: absolute;
  top: 3.5rem;
}
#notification {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 20px;
  position: absolute;
  top: 6rem;
  width: 100%;
}
@media screen and (min-width: 1024px) {
  #player {
    position: absolute;
    top: 0.5rem;
    left: 0.5rem;
    width: 200px;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    padding: 0.5rem;
  }
  #board {
    height: 600px;
    max-width: 600px;
  }
  .card {
    height: 200px;
    width: 200px;
  }
  .icon {
    width: 50px;
    height: 50px;
  }
  #notification {
    position: absolute;
    top: 0.5rem;
    width: 100%;
    display: flex;
    justify-content: center;
    font-size: 48px;
  }
  #btSlaySword {
    top: 0.5rem;
    left:13.5rem;
  }
}
</style>
