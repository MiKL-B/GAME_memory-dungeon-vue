<template>
  <div class="background"></div>
  <div v-if="currentScreen === 'main'" id="mainScreen">
    <h1 id="gameTitle">Memory Dungeon</h1>
    <button class="btScreen" @click="startGame">Play</button>
    <button class="btScreen" @click="displayCredits">Credits</button>
  </div>

  <div v-if="currentScreen === 'game'" id="gameScreen">
    <div class="statusBar">
      <p>Count: {{ count }}</p>
    </div>
    <div id="board">
      <div
        class="card"
        v-for="card in cards"
        @click="flip(card)"
        :class="card.flipped ? 'flipped' : 'unflipped'"
      >
        <div v-if="card.flipped === false" class="face">
          <img class="card-img" src="/dungeon.png" />
        </div>
        <div v-else class="back">
          <img
            class="card-img"
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
      <p class="paragraphConfirm" v-html="formattedMessageConfirm"></p>
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
      message: "",
      // modal confirm
      isModalConfirmVisible: false,
      messageConfirm: "",
      isValidateConfirm: false,
      cards: [],
      count: 0,
      time: 0,
    };
  },
  mounted() {
    const imagesToPreload = [
      "/beer.png",
      "/brain.png",
      "/coins.png",
      "/dungeon.png",
      "/joker.png",
      "/monster.png",
      "/potion.png",
      "/rune-stone.png",
      "/shield.png",
      "/sword.png",
    ];

    this.preloadImages(imagesToPreload);
  },

  computed: {
    formattedMessageConfirm() {
      return this.messageConfirm.replace(/\r\n/g, "<br>");
    },
  },

  methods: {
    preloadImages(images) {
      images.forEach((src) => {
        const img = new Image();
        img.src = src;
      });
    },

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
    startMainScreen() {
      let msg = "Bienvenue dans Memory Dungeon.\r\n\r\n";
      msg +=
        "Parcourez le donjon, trouvez des paires, ramassez les épées afin de vaincre les monstres sans difficultés, sinon il vous en coûtera.\r\n\r\n";
      msg += "Serez-vous assez courageux pour obtenir les 10 pièces d'or ?";

      this.showModalConfirm(msg).then((result) => {
        if (result) {
          this.showScreen("main");
        }
      });
    },
    startGame() {
      this.resetBoard();
      this.resetPlayer();
      this.generateRandomBoard();
      this.shuffleArray(this.cards);
      this.showScreen("game");
      this.flippedBackCards(true);
      setTimeout(() => {
        this.flippedBackCards(false);
      }, 3000);
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
        {
          name: "joker",
          color: "green",
        },
        {
          name: "brain",
          color: "brain",
        },
        {
          name: "rune-stone",
          color: "cyan",
        },
      ];
      let names = [
        "potion",
        "monster",
        "coins",
        "sword",
        "shield",
        "beer",
        "joker",
        "brain",
        "rune-stone"
      ];
      let selectedCards = [];

      while (selectedCards.length < items.length) {
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
    },
    resetBoard() {
      this.cardsToCompare = [];
      this.flippedCards = 0;
      this.flippedBackCards(true);
      this.count = 0;
    },
    resetPlayer() {
      this.attack = 0;
      this.currentRoom = 1;
      this.gold = 0;
      this.filledHearts = 3;
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
      if (this.checkIfAllCardsFlipped()) {
        this.winGame();
        return;
      }
      if (this.soundStopSFX === false) {
        this.playSoundFlipCard();
      }
      this.cardsToCompare.push(card);
      this.compareCard();
    },
    flippedBackCards(isFlipped) {
      for (let i = 0; i < this.cards.length; i++) {
        this.cards[i].flipped = isFlipped;
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

      this.count++;
      if (card1.name !== card2.name) {
        setTimeout(() => {
          card1.flipped = false;
          card2.flipped = false;

          this.cardsToCompare = [];
          this.flippedCards = 0;
        }, 1000);
        return;
      }

      this.cardsToCompare = [];
      this.flippedCards = 0;
    },

    displayCredits() {
      let msg = "Images:\r\n";
      msg += "Source: https://game-icons.net/\r\n\r\n";
      this.showModalConfirm(msg).then((result) => {
        if (result) {
          return;
        }
      });
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
  animation: smooth 0.5s ease-in-out;
}


.statusBar {
  display: flex;
  justify-content: center;
}
@keyframes smooth {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
.btScreen {
  width: 150px;
  padding: 0.25rem 0.75rem;
}
#gameTitle {
  font-size: 48px;
}

#gameScreen {
  display: flex;
  justify-content: center;
  flex-direction: column;
  height: 100vh;
  animation: smooth 0.5s ease-in-out;
}

#board {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  gap: 0.5rem;
  margin: auto;
}
.card {
  background: var(--background);
  transform-style: preserve-3d;
}
.card-img {
  height: 100px;
  width: 100px;
  border: 2px solid var(--grey);
}
.paragraphConfirm {
  font-size: 1rem;
}
.unflipped {
  animation: unflip 0.5s;
  transform: rotateY(0deg);
}
@keyframes unflip {
  from {
    -webkit-transform: rotateY(180deg);
    transform: rotateY(180deg);
  }
  to {
    -webkit-transform: rotateY(0deg);
    transform: rotateY(0deg);
  }
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
  height: 100px;
  backface-visibility: hidden;
  transform: rotateY(0deg);
}

.back {
  background: var(--background);
  height: 100px;
  backface-visibility: hidden;
  transform: rotateY(180deg);
}

p {
  font-size: 28px;
}
.background {
  position: absolute;
  top: -30%;
  height: 200%;
  width: 150%;
  opacity: 0.1;
  z-index: -1;
  background: url("/dungeon.png");
  background-repeat: repeat;
}
@media screen and (min-width: 1024px) {
  #board {
    grid-template-columns: repeat(5, 200px);
  }
  .card-img {
    height: 200px;
    width: 200px;
  }

  .face,
  .back {
    height: 200px;
  }
}


</style>
