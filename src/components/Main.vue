<template>
  <div class="hello">
    <h1>L.U.N.A.L.A</h1>
    <p>
      The Logographic Uncluttered Natural Aquisition Language Achiever
    </p>

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
      <b-button class="button" @click="getVocab">Test</b-button> 
    </div>
    
  </div>
</template>

<script>
import axios from 'axios'

const NUM_HEADERS = 6;

export default {
  name: 'Main',
  props: {
    msg: String
  },
  data() {
    return {
      vocab_file: [],
      file_input: [],
    }
  },
  methods: {
    getVocab() {
      let path = 'http://localHost:5000/getVocab'
      axios.get(path)
        .then((res) => {
          console.log(res.data)
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
              Kanji, Hiragana, English, Origin, Difficulty, and New")
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
        this.getVocab();
      }
    }
  }, 
  beforeMount() {
    this.getVocab();
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
