<template>
  <div class="hello">
    <h1>L.U.N.A.L.A</h1>
    <p>
      The Logographic Uncluttered Natural Aquisition Language Achiever
    </p>

    <b-form-file
      class="file_in"
      v-model="file1"
      :state="Boolean(file1)"
      accept=".csv"
      placeholder="Choose a replacement CSV here..."
      drop-placeholder="Drop file here..."
    ></b-form-file>
    <div class="mt-3">Selected file: {{ file1 ? file1.name : '' }}</div>
    
  </div>
</template>

<script>
export default {
  name: 'Main',
  props: {
    msg: String
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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
