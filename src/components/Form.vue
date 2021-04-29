<template>
  <div>
    <div>
      <button class="btn btn-info" @click="onPickFile">Upload</button>
    </div>
    <div>
      <input
          type="file"
          style="display:none"
          ref="fileInput"
          accept="application/json"
          @change="onFilePicked"/>
      <textarea id="tree" name="tree" rows="20" cols="100" v-model="treeJson"/>
    </div>
    <div>
      <Dagre :tree="treeJson" v-if="treeJson"/>
    </div>
  </div>
</template>

<script>
import Dagre from "./Dagre";

export default {
  name: 'Visualizer',
  components: {
    Dagre
  },
  data:
      function () {
        return {
          treeJson: ""
        }
      },
  methods: {
    onPickFile() {
      this.$refs.fileInput.click()
    },
    onFilePicked(event) {
      const files = event.target.files
      const fileReader = new FileReader()
      fileReader.addEventListener('load', () => {
        this.treeJson = fileReader.result
      })
      fileReader.readAsText(files[0]);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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
