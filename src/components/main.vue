<template>
  <div class="main">
    <h1>Крестики - нолики</h1>
    <p>Старт / сброс: Новая игра. Правая кнопка мыши - 0, левая - крест</p>
    <areacomp />
  </div>
</template>

<script>
import areacomp from './area.vue'

export default {
  name: 'maincomp',
  components: {
    areacomp,
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
