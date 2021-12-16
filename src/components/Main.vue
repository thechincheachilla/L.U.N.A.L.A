<template>
  <div class="hello">
    <h1>L.U.N.A.L.A</h1>
    <p>
      The Logographic Uncluttered Natural Aquisition Language Achiever
    </p>
    <div v-if="dataLoaded">
      <b-card-group deck v-for="row in cardGrid" :key="cardGrid.indexOf(row)" class="cards"> 
        <b-card v-for="item in row" :key="item" class="card" text-variant="white" :header='getDifficulty(vocab_file["Difficulty"][item])'>
          <b-card-text class="card_text">{{vocab_file["Kanji"][item]}}</b-card-text>
        </b-card>
      </b-card-group>
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
      <b-button class="button" @click="replaceCSV">Submit</b-button> 
    </div>
    
  </div>
</template>

<script>
import axios from 'axios'

const NUM_HEADERS = 5;
const INIT_ROWS = 6;
const CARDS_PER_ROW = 6;

export default {
  name: 'Main',
  props: {
    msg: String
  },
  data() {
    return {
      vocab_file: [],
      file_input: [],
      cards: [],
      cardGrid: [],
      dataLoaded: false,
    }
  },
  methods: {
    async getVocab() {
      let path = 'http://localHost:5000/getVocab'
      await axios.get(path)
        .then((res) => {
          this.vocab_file = res.data;
          console.log(this.vocab_file)
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
      console.log(headers)
      if (headers.length != NUM_HEADERS) {
        alert("CSV File not correctly formatted. \n\n\
              The CSV should contain columns in the order of: \n\
              Kanji, Hiragana, English, Origin, and Difficulty")
        this.file_input = [];
      }
    },
    async replaceCSV() {
      if (this.file_input != []) {
        let formData = new FormData();
        formData.append('file', this.file_input);
        let path = 'http://localHost:5000/replace'
        await axios.post(path, formData, { 
          headers: {
            'Content-Type': 'multipart/form-data' 
          }
        }).then(function() {
          console.log('File successfully sent');
        }).catch(function() {
          console.log('File failed to send')
        });
        await this.getVocab();
        this.initCards();
      }
    },
    initCards() {
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
      let numSet = [];
      for (let i = 0; i < numCards; i++) {
        numSet.push(i);
      }
      numSet = shuffle(numSet);
      console.log(numSet);
      this.cardGrid = [];
      for (let i = 0; i < INIT_ROWS; i++) {
        let currRow = [];
        for (let j = 0; j < CARDS_PER_ROW; j++) {
          try {
            currRow.push(numSet[CARDS_PER_ROW*i+j]);
          }
          catch (error) {
            alert("Not enough cards in the deck");
          }
        }
        this.cardGrid.push(currRow);
      }
      console.log(this.cardGrid);
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
    }
  }, 
  async beforeMount() {
    await this.getVocab();
    this.initCards();
    this.dataLoaded = true; 
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.button {
  margin-top: 20px;
  background-color:rgb(46, 24, 145)
}
.file_in {
  width: 20%;
  text-align: left;
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
