<template>
  <div class="background"></div>
  <div v-if="currentScreen === 'title'" id="titleScreen">
    <h1 id="gameTitle">Memory Dungeon</h1>
    <button class="btScreen" @click="startMainScreen">Play</button>
    <button class="btScreen" @click="showScreen('option')">Options</button>
    <button class="btScreen" @click="displayCredits">Credits</button>
  </div>
  <div v-if="currentScreen === 'main'" id="mainScreen">
    <h1>Lobby</h1>
    <button class="btScreen" @click="startGame">Launch</button>
    <button class="btScreen" @click="showScreen('title')">Quit</button>
    <!-- <button class="btScreen" @click="showScreen('skills')">Skills</button>
    <button class="btScreen" @click="showScreen('cards')">Cards</button>
    <button class="btScreen" @click="showScreen('achievements')">
      Achievements
    </button> -->
    <span>Coming soon...</span>
    <button class="btScreen disabled">Skills</button>
    <button class="btScreen disabled">Cards</button>
    <button class="btScreen disabled">Achievements</button>
  </div>
  <div v-if="currentScreen === 'skills'" id="skillsScreen">
    <h1 class="center">Skills</h1>
    <div class="skills">
      <h2>Memory</h2>
      <div v-for="i in 10" class="skill">
        <span>Skill name</span>
        <div class="skillAction">
          <button>+</button>
          <span>75</span>
          <img class="imgMenu" src="/brain.png" alt="" />
        </div>
      </div>
      <h2>Rune</h2>
      <div v-for="i in 10" class="skill">
        <span>Skill name</span>
        <div class="skillAction">
          <button>+</button>
          <span>75</span>
          <img class="imgMenu" src="/rune-stone.png" alt="" />
        </div>
      </div>
    </div>
    <button @click="showScreen('main')" class="center btScreen">Back</button>
  </div>
  <div v-if="currentScreen === 'cards'" id="cardsScreen">
    <h1 class="center">Cards</h1>
    <div class="cards">
      <div v-for="i in 10" class="cardToBuy">
        <span>Cards name</span>
        <div class="cardAction">
          <button>Buy</button>
          <span>75</span>
          <img class="imgMenu" src="/coins.png" alt="" />
        </div>
      </div>
    </div>
    <button @click="showScreen('main')" class="center btScreen">Back</button>
  </div>
  <div v-if="currentScreen === 'achievements'" id="achievementScreen">
    <h1 class="center">Achievements</h1>
    <div class="achievements">
      <div v-for="i in 10" class="achievement">
        <img class="imgMenu" src="/help.png" />
        <span>Achievement name</span>
      </div>
    </div>
    <button @click="showScreen('main')" class="center btScreen">Back</button>
  </div>
  <div v-if="currentScreen === 'game'" id="gameScreen">
    <div id="player">
      <div class="field">
        <!-- <img
          class="icon"
          v-for="index in hearts"
          :key="index"
          :src="getHeartImage(index)"
        /> -->
        <div class="field">
          <img class="icon" src="/heart-red.png" />
          <p>{{ hearts }}</p>
        </div>
      </div>
      <div class="field">
        <img class="icon" src="/dungeon.png" />
        <p>{{ currentRoom }}</p>
      </div>

      <div class="field">
        <img class="icon" src="/coins.png" />
        <p>{{ gold }}</p>
      </div>
      <div class="field">
        <img class="icon" src="/brain.png" />
        <p>0</p>
      </div>
      <div class="field">
        <img class="icon" src="/rune-stone.png" />
        <p>0</p>
      </div>
    </div>

    <div id="notification">
      {{ message }}
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
          <!-- <span class="card-attack" v-if="card.attack">
            {{ card.attack }}
          </span> -->
        </div>
      </div>
    </div>
    <div class="bottomContainer">
      <button
        id="btSlaySword"
        v-if="attack > 0 && flippedCards === 0"
        @click="slayMonster"
        class="primary"
      >
        <img src="/sword.png" />
      </button>
      <button v-else id="btSlaySword" class="disabled">
        <img src="/sword-grey.png" />
      </button>
      <!-- <button class="disabled">
        <img src="/cart.png" alt="" />
      </button>
      <button class="disabled">
        <img src="/trophy.png" alt="" />
      </button>
      <button class="disabled">
        <img src="/cog.png" alt="" />
      </button> -->
    </div>
  </div>
  <div v-if="currentScreen === 'option'" id="optionScreen">
    <label for="volumeControl">Music</label>
    <input
      id="volumeControl"
      type="range"
      min="0"
      max="1"
      step="0.01"
      v-model="volume"
      @input="updateVolume"
    />
    <button @click="toggleSound">
      <img v-if="soundStop === false" src="/speaker.png" alt="" />
      <img v-else src="/speaker-off.png" alt="" />
    </button>
    <label for="volumeControlSFX">SFX</label>
    <input
      id="volumeControlSFX"
      type="range"
      min="0"
      max="1"
      step="0.01"
      v-model="volumeSFX"
      @input="updateVolumeSFX"
    />
    <button @click="toggleSoundSFX">
      <img v-if="soundStopSFX === false" src="/speaker.png" alt="" />
      <img v-else src="/speaker-off.png" alt="" />
    </button>
    <button @click="showScreen('title')">Back</button>
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
import { Howl } from "howler";
export default {
  name: "App",
  components: {
    ModalConfirm,
  },
  data() {
    return {
      sound: null,
      soundStop: true,
      volume: 0.5,
      volumeSFX: 0.5,
      soundStopSFX: true,
      soundFlipCard: null,
      currentScreen: "title",
      flippedCards: 0,
      cardsToCompare: [],
      gold: 0,
      hearts: 3,
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
    const imagesToPreload = [
      "/beer.png",
      "/coins.png",
      "/dungeon.png",
      "/sword.png",
      "/sword-grey.png",
      "/heart-red.png",
      "/heart-grey.png",
      "/speaker.png",
      "/speaker-off.png",
      "/ladder.png",
      "/monster.png",
      "/potion.png",
      "/shield.png",
      "Dungeon-Exploration Music 1.ogg",
    ];

    this.preloadImages(imagesToPreload);
    if (this.sound) {
      this.sound.volume(this.volume);
    }
    if (this.soundSFX) {
      this.soundSFX.volume(this.volumeSFX);
    }
  },
  beforeDestroy() {
    if (this.sound) {
      this.sound.unload();
    }
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

    toggleSound() {
      this.soundStop = !this.soundStop;
      if (this.soundStop) {
        this.stopSound();
      } else {
        this.playSound();
      }
    },
    playSound() {
      this.sound = new Howl({
        src: ["Dungeon-Exploration Music 1.ogg"],
        volume: this.volume,
        loop: true,
        html5: true,
        autoplay: true,
      });
      this.sound.play();
    },
    stopSound() {
      if (this.sound) {
        this.sound.stop();
      }
    },
    updateVolume() {
      if (this.sound) {
        this.sound.volume(this.volume);
      }
    },

    playSoundFlipCard() {
      this.soundFlipCard = new Howl({
        src: ["flipcard-91468.mp3"],
        volume: this.volumeSFX,
        loop: false,
        html5: true,
        autoplay: false,
      });
      this.soundFlipCard.play();
    },
    stopSoundFlipCard() {
      if (this.soundFlipCard) {
        this.soundFlipCard.stop();
      }
    },
    updateVolumeSFX() {
      if (this.soundSFX) {
        this.soundSFX.volume(this.volumeSFX);
      }
    },
    toggleSoundSFX() {
      this.soundStopSFX = !this.soundStopSFX;
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
          // attack: 1,
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
      ];
      let names = [
        "potion",
        "monster",
        "coins",
        "sword",
        "shield",
        "beer",
        "joker",
      ];
      let selectedCards = [];

      while (selectedCards.length < 4) {
        let index = Math.floor(Math.random() * items.length);

        let randomCard = {
          name: items[index].name,
          flipped: false,
          img: names[index],
          color: items[index].color,
          // attack: items[index].attack ,
        };

        if (!this.cardExists(randomCard, selectedCards)) {
          selectedCards.push(randomCard);
        }
        //  selectedCards.push(randomCard);
        // console.log(selectedCards)
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
      if (this.soundStopSFX === false) {
        this.playSoundFlipCard();
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
          if (this.soundStopSFX === false) {
            this.playSoundFlipCard();
          }
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

      // document.body.classList.add("shake");
      // setTimeout(() => {
      //   document.body.classList.remove("shake");
      // }, 1000);

      this.attack = 0;
    },
    showMessage(text) {
      this.message = text;
      setTimeout(() => {
        this.message = "";
      }, 2000);
    },

    increaseHearts() {
      this.hearts++;
      if (this.hearts >= 3) {
        this.hearts = 3;
      }
      this.showMessage(`You have earned a health point!`);
    },
    decreaseHearts() {
      this.hearts--;
      if (this.hearts === 0) {
        this.hearts = 0;
      }
      this.showMessage("You have lost a health point!");
    },

    displayCredits() {
      let msg = "Images:\r\n";
      msg += "Source: https://game-icons.net/\r\n\r\n";
      msg += "SFX:\r\n";
      msg +=
        "Source: https://pixabay.com/sound-effects/search/card-flip/\r\n\r\n";
      msg += "Music:\r\n";
      msg +=
        "Source: https://muhammadriza.itch.io/free-fantasy-rpg-music-pack\r\n\r\n";
      msg +=
        "Music by JP Soundworks (https://www.youtube.com/c/JPSoundworks/)\r\n\r\n";
      msg +=
        "Music by JP Soundworks, Pack Published by Platonic Game Studio.\r\n\r\n";
      msg += "(C) 2019 - 2020 JP Soundoworks\r\n";
      msg += "(P) 2019 - 2020 Platonic Game Studio\r\n";

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
#titleScreen,
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

.achievement {
  gap: 0.5rem;
  display: flex;
  padding: 0 0.5rem;
  justify-content: space-between;
  align-items: center;
}
.imgMenu {
  width: 50px;
  height: 50px;
}
.center {
  display: flex;
  justify-content: center;
  margin: 0 auto;
}
#skillsScreen,
#cardsScreen,
#achievementScreen {
  animation: smooth 0.5s ease-in-out;
}
.skills,
.cards,
.achievements {
  overflow: scroll !important;
  height: calc(100vh - 6rem);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  max-width: 300px;
  margin: auto;
}
.skill,
.cardToBuy {
  display: flex;
  padding: 0 0.5rem;
  justify-content: space-between;
  align-items: center;
}
.skillAction,
.cardAction {
  display: flex;
  align-items: center;
  gap: 0.5rem;
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
/* .card-attack {
  position: absolute;
  bottom: 0.5rem;
  right: 0.5rem;
  font-size: 24px;
  filter: drop-shadow(1px 1px 2px black);
} */

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
  display: flex;
  justify-content: space-around;
  gap: 0.1rem;
  margin: 0.5rem;
}
.field {
  gap: 0 0.2rem;
}
p {
  font-size: 28px;
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
/* .shake {
  animation: shake 0.2s;
}
@keyframes shake {
  0% {
    transform: translate(6px, 0);
  }
  25% {
    transform: translate(-6px, 0);
  }
  50% {
    transform: translate(6px, 0);
  }
  75% {
    transform: translate(-6px, 0);
  }
  100% {
    transform: translate(0, 0);
  }
} */
/* .validation {
  animation: validation 0.5s forwards;
}
@keyframes validation {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
} */
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
    margin: 0;
  }
  .skills,
  .cards {
    max-width: 600px;
    margin: auto;
  }
  #board {
    grid-template-columns: repeat(3, 200px);
  }
  .card-img {
    height: 200px;
    width: 200px;
  }
  .icon {
    width: 50px;
    height: 50px;
  }
  .face,
  .back {
    height: 200px;
  }

  #notification {
    position: absolute;
    top: 0.5rem;
    width: 100%;
    display: flex;
    justify-content: center;
    font-size: 48px;
  }
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
.bottomContainer {
  position: absolute;
  left: 0.5rem;
  right: 0.5rem;
  bottom: 0.5rem;
  display: flex;
  justify-content: center;
  gap: 0.5rem;
}
.bottomContainer button {
  width: 50px;
  height: 50px;
  padding: 0.5rem;
}

/* optionscreen */
#optionScreen {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100vh;
  padding: 0.5rem;
  gap: 1rem;
}
#optionScreen button {
  width: 150px;
  padding: 0.25rem 0.75rem;
}
</style>
