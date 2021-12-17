<template>
  <div class="hello">

    <h1 style="font-weight:bold; color:rgb(46, 24, 145);">L.U.N.A.L.A</h1>

    <p style="margin-bottom:40px">
      The Logographic Uncluttered Natural Aquisition Language Achiever
    </p>

    <b-form-group v-slot="{ ariaDescribedby }">
      <b-form-checkbox-group
        v-model="selected_diff"
        :options="difficulties"
        :aria-describedby="ariaDescribedby"
        name="flavour-1a"
        @change="updateChecklist()"
      ></b-form-checkbox-group>
    </b-form-group>

    <b-form-group v-slot="{ ariaDescribedby }">
      <b-form-checkbox-group
        v-model="selected_cards"
        :options="numCards"
        :aria-describedby="ariaDescribedby"
        name="flavour-1a"
        @change="updateChecklist()"
      ></b-form-checkbox-group>
    </b-form-group>

    <b-button class="button" 
      style="margin-bottom:40px; background-color:rgb(211, 209, 1); color:rgb(46, 24, 145);" 
      @click="generateCards()"
      v-b-tooltip.hover.left title="Generate a new shuffled batch of cards with the above selection fields."
      data-toggle="tooltip"
      >Generate Cards</b-button>

    <b-button class="button" 
      style="margin-bottom:40px; background-color:rgb(211, 209, 1); color:rgb(46, 24, 145);" 
      @click="setDifficulties()"
      v-b-tooltip.hover.right title="Toggle on/off whether clicking a card changes the language or the difficulty."
      data-toggle="tooltip"
      >Update Difficulties</b-button>

    <div v-if="dataLoaded" class="card_div">
      <b-card-group deck v-for="row in cardGrid" :key="cardGrid.indexOf(row)" class="cards"> 
        <b-card v-for="item in row" 
          :key="item" 
          class="card" 
          text-variant="white" 
          :header='getDifficulty(vocab_file["Difficulty"][item])' 
          header-bg-variant="info"
          >
          <b-card-text class="card_text">{{getText(item)}}</b-card-text>
          <button class="stretched-link" 
            style="height:0px; background-color:rgb(46, 24, 145); border-style:none" 
            @click="cycleCard(item)"></button>
        </b-card>
      </b-card-group>
      <b-button class="button" style="margin-bottom:40px; background-color:rgb(211, 209, 1); color:rgb(46, 24, 145);" @click="getCards()">Next Cards</b-button>
    </div>

    <div>
      <b-button variant="info" style="margin-bottom:40px; margin-top:40px" @click="saveChanges()">Save Changes</b-button> 
    </div>

    <b-form-file
      class="file_in"
      v-model="file_input"
      :state="Boolean(file_input)"
      accept=".csv"
      placeholder="Choose a replacement CSV here..."
      drop-placeholder="Drop file here..."
      @change="loadCSV"
    ></b-form-file>
    <div>
      <b-button class="button" style="margin-bottom:40px" @click="replaceCSV">Submit</b-button> 
    </div>
    
  </div>
</template>

<script>
import axios from 'axios'

const NUM_HEADERS = 5;
const CARDS_PER_ROW = 7;

export default {
  name: 'Main',
  data() {
    return {
      vocab_file: [],
      file_input: [],
      cards: [],
      offset: 0,
      numSet: [],
      cardGrid: [],
      cardStates: {},
      dataLoaded: false,
      difficulties: ["New", "Easy", "Medium", "Hard"],
      numCards: [14, 35, 70, 140, 280, 469],
      selected_diff: ["New", "Easy", "Medium", "Hard"],
      selected_cards: [35],
      difficultyMode: false
    }
  },
  methods: {
    async getVocab() {
      let path = 'http://localHost:5000/getVocab'
      await axios.get(path)
        .then((res) => {
          this.vocab_file = res.data;
          console.log('Vocab data:', this.vocab_file);
        })
        .catch(() => {
          alert("API connection failed; please input your own CSV");
        })
    },
    loadCSV(ev) {
      const newFile = ev.target.files[0]
      const reader = new FileReader();
      reader.readAsText(newFile);
      reader.onload = e => this.checkData(e.target.result);
    },
    checkData(file) {
      let fileAsLines = file.split("\n");
      let headers = fileAsLines[0].split(",");
      headers[headers.length-1] = headers[headers.length-1].replace(/^[\r\n]+|\.|[\r\n]+$/g, "");
      console.log('Headers:', headers)
      if (headers.length != NUM_HEADERS) {
        alert("CSV File not correctly formatted. \n\n\
              The CSV should contain columns in the order of: \n\
              Kanji, Hiragana, English, Origin, and Difficulty")
        this.file_input = [];
      }
    },
    async replaceCSV() {
      console.log("FILE", this.file_input)
      if (this.file_input != []) {
        let formData = new FormData();
        formData.append('file', this.file_input);
        let path = 'http://localHost:5000/replace'
        await axios.post(path, formData, { 
          headers: {
            'Content-Type': 'multipart/form-data' 
          }
        }).then(() => {
          console.log('File successfully sent');
        }).catch((error) => {
          alert('File failed to send');
          console.log("File send error:", error)
        });
        await this.getVocab();
        this.selected_diff = ["New", "Easy", "Medium", "Hard"];
        this.selected_cards = [36];
        this.shuffleCards();
        this.generateCards(0);
      }
    },
    shuffleCards() {
      let shuffle = function(arr) {
        let currIndex = arr.length, randomIndex;
        while (currIndex !== 0) {
          randomIndex = Math.floor(Math.random() * currIndex);
          currIndex--;
          [arr[currIndex], arr[randomIndex]] = [arr[randomIndex], arr[currIndex]];
        }
        return arr;
      }
      let numCards = Object.keys(this.vocab_file['Kanji']).length;
      this.numSet = [];
      for (let i = 0; i < numCards; i++) {
        if (this.selected_diff.includes(this.getDifficulty(this.vocab_file['Difficulty'][i]))) {
          this.numSet.push(i);
        }
      }
      this.numSet = shuffle(this.numSet);
      //console.log(this.numSet.length, this.numSet)
    },
    generateCards() {
      this.offset = 0;
      this.shuffleCards();
      this.getCards(this.offset);
    },
    getCards() {
      if (this.offset <= Object.keys(this.vocab_file['Kanji']).length) {
        window.scrollTo(0, 400);
        let breakEarly = false;
        // console.log(numSet);
        this.cardGrid = [];
        let numRows = this.selected_cards[0]/CARDS_PER_ROW;
        for (let i = 0; i < numRows; i++) {
          if (breakEarly) {
            break;
          }
          let currRow = [];
          for (let j = 0; j < CARDS_PER_ROW; j++) {
            try {
              currRow.push(this.numSet[CARDS_PER_ROW*i+j+this.offset]);
              this.cardStates[this.numSet[CARDS_PER_ROW*i+j+this.offset].toString()] = 0;
            }
            catch (error) {
              console.log("Not enough cards in the deck");
              breakEarly = true;
              currRow.pop();
              break;
            }
          }
          this.cardGrid.push(currRow);
          // console.log(this.cardGrid)
        }
        this.offset += this.selected_cards[0];
        // console.log(this.cardGrid);
        // console.log(this.cardStates);
      }
    },
    getDifficulty(num) {
      if (num === 0) {
        return "New"
      }
      if (num === 1) {
        return "Easy"
      }
      if (num === 2) {
        return "Medium"
      }
      if (num === 3) {
        return "Hard"
      }
      if (num === "New") {
        return 1
      }
      if (num === "Easy") {
        return 1
      }
      if (num === "Easy") {
        return 2
      }
      if (num === "Easy") {
        return 3
      }
    },
    getText(item) {
      if (this.cardStates[item] === 0) {
        return this.vocab_file["Kanji"][item];
      }
      if (this.cardStates[item] === 1) {
        return this.vocab_file["Hiragana"][item];
      }
      return this.vocab_file["English"][item];
    },
    cycleCard(item) {
      if (!this.difficultyMode) {
        this.cardStates[item] = this.cardStates[item] + 1; 
        if (this.cardStates[item] > 2) {
          this.cardStates[item] = 0;
        }
        // console.log(this.cardStates[item]);
      }
      else {
        console.log(this.vocab_file["Difficulty"])
        if (this.vocab_file["Difficulty"][item] < 3) {
          this.vocab_file["Difficulty"][item] = this.vocab_file["Difficulty"][item] + 1;
        }
        else {
          this.vocab_file["Difficulty"][item] = 0;
        }
      }
      this.$forceUpdate();
    },
    setDifficulties() {
      this.difficultyMode = !this.difficultyMode;
    },
    updateChecklist() {
      let currCard = this.selected_cards[this.selected_cards.length-1];
      this.selected_cards = [];
      this.selected_cards.push(currCard);
      // console.log(this.selected_diff)
      // console.log(this.selected_cards)
    },
    async saveChanges() {
      let path = 'http://localHost:5000/saveCSV'
      axios.post(path, this.vocab_file)
        .then(() => {
          console.log("File successfully sent");
        })
        .catch((error) => {
          alert("File failed to send");
          console.log("File send error:", error);
        })
    }
  }, 
  async beforeMount() {
    await this.getVocab();
    this.shuffleCards();
    this.generateCards(0);
    this.dataLoaded = true; 
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.button {
  margin-left: 10px;
  margin-right: 10px;
  margin-top: 20px;
  margin-bottom: 40px;
  background-color:rgb(46, 24, 145);
  border:none;
} 
.file_in {
  width: 20%;
  text-align: left;
  margin-top: 30px;
}
.card_div {
  /* overflow-y: auto; 
  height: 1000px; */
}
.cards {
  margin: auto;
  width: 80%;
  margin-bottom: 20px;
}
.card {
  background-color:rgb(46, 24, 145);
}
.card_text {
  font-size: 30px;
  color:rgb(255, 255, 255);
}
h1 {
  margin: 20px 0 10px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
