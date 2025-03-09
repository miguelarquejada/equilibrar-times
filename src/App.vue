<script setup>
import {ref} from 'vue'

import Principal from './Principal.vue';
import Final from './Final.vue';

const exibirResultado = ref(false);
const timesResultado = ref([]);
</script>

<template>
  <div id="principal">
    <Principal v-if="timesResultado.length == 0" @montar-times="v => timesResultado = v" />
    <Final v-else :times="timesResultado" @voltar="v => timesResultado = []" />
  </div>
</template>

<style scoped>
#principal {
  margin: 40px 20px;
}
</style>
