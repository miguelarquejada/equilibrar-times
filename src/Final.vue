<script setup>
import {ref} from 'vue'

const props = defineProps({
    times: Array
});

const emit = defineEmits(['voltar']);

const numeroTime = ref(0);

const niveis = [
    { id: 1, icone: '👶', descricao: 'primeira vez' },
    { id: 4, icone: '🏄‍♂️', descricao: 'tirando onda' },
    { id: 2, icone: '👣', descricao: 'engatinhando' },
    { id: 5, icone: '🦸‍♂️', descricao: 'super-man' },
    { id: 3, icone: '🕺', descricao: 'pegando o jeito' },
    { id: 6, icone: '🥋', descricao: 'ninja' },
];

const obterIconePorNivel = nivel => {
    return niveis.find(n => n.id === nivel).icone;
}

const voltar = () => {
    numeroTime.value = 0;
    emit('voltar', 0);
}
</script>

<template>
  <div class="bold texto-roxo">Times equilibrados</div>
  <div class="mt-20" v-for="(time, index) in times">
    <div class="bold">Time {{ index+1  }}</div>
    <div class="mt-10" v-for="jogador in time">
        <div>{{ obterIconePorNivel(jogador.nivel) }} {{ jogador.nome }}</div>
    </div>
  </div>

  <button @click="voltar" class="botao-primario mt-20">Voltar</button>
</template>

<style scoped>

</style>
