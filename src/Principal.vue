<script setup>
import { ref } from 'vue'

const emit = defineEmits(['montarTimes'])

const niveis = [
    { id: 1, icone: '👶', descricao: 'primeira vez' },
    { id: 4, icone: '🏄‍♂️', descricao: 'tirando onda' },
    { id: 2, icone: '👣', descricao: 'engatinhando' },
    { id: 5, icone: '🦸‍♂️', descricao: 'super-man' },
    { id: 3, icone: '🕺', descricao: 'pegando o jeito' },
    { id: 6, icone: '🥋', descricao: 'ninja' },
];

const jogadores = ref([]);

const novoJogador = ref({
    nome: '',
    nivel: 0
});

const indexJogadorSendoAtualizado = ref('-1');

const numeroPessoasPorTime = ref('');

const selecionarNivel = id => {
    novoJogador.value.nivel = id;
}

const reiniciarNovoJogador = () => {
    novoJogador.value = {
        nome: '',
        nivel: 0
    };
    indexJogadorSendoAtualizado.value = '-1';
}

const novoJogadorValido = () => novoJogador.value.nome.trim() !== '' && novoJogador.value.nivel !== 0;

const estaAtualizando = () => indexJogadorSendoAtualizado.value !== '-1';

const adicionarJogador = () => {
    if (!novoJogadorValido()) {
        return;
    }

    if (estaAtualizando()) {
        jogadores.value[indexJogadorSendoAtualizado.value] = novoJogador.value;
    } else {
        jogadores.value.push(novoJogador.value);
    }
    reiniciarNovoJogador();
}

const removerJogador = () => {
    jogadores.value = jogadores.value.filter((el, index) => index !== indexJogadorSendoAtualizado.value);
    reiniciarNovoJogador();
}

const obterIconePorNivel = nivel => {
    return niveis.find(n => n.id === nivel).icone;
}

const selecionarJogador = (jogador, index) => {
    indexJogadorSendoAtualizado.value = index;
    novoJogador.value = { ...jogador };
}

const montarTimes = () => {
    debugger;
    if (estaAtualizando() || jogadores.value.length < 2 || numeroPessoasPorTime.value < 2) {
        return;
    }

    const jogadoresPorNivel = jogadores.value.reduce((acc, jogador) => {
        acc[jogador.nivel] = acc[jogador.nivel] || [];
        acc[jogador.nivel].push(jogador);
        return acc;
    }, {});
    
    // Embaralhar os jogadores dentro de cada nível
    for (let nivel in jogadoresPorNivel) {
        jogadoresPorNivel[nivel] = jogadoresPorNivel[nivel].sort(() => Math.random() - 0.5);
    }

    // Criar times intercalando jogadores do nível mais alto e do mais baixo
    const times = [];
    let niveisOrdenados = Object.keys(jogadoresPorNivel).map(n => parseInt(n)).sort((a, b) => a - b);
    let jogadoresRestantes = jogadores.value.length;
    
    while (jogadoresRestantes > 0 && niveisOrdenados.length > 0) {
        let nivelMaisBaixo = niveisOrdenados[0];
        let nivelMaisAlto = niveisOrdenados[niveisOrdenados.length - 1];
        
        for (let nivel of [nivelMaisBaixo, nivelMaisAlto]) {
            if (jogadoresPorNivel[nivel] && jogadoresPorNivel[nivel].length > 0) {
                if (times.length === 0 || times[times.length - 1].length === numeroPessoasPorTime.value) {
                    times.push([]);
                }
                times[times.length - 1].push(jogadoresPorNivel[nivel].pop());
                jogadoresRestantes--;
                
                // Remover o nível da lista se não houver mais jogadores nele
                if (jogadoresPorNivel[nivel].length === 0) {
                    niveisOrdenados = niveisOrdenados.filter(n => n !== nivel);
                }
            }
        }
    }

    // Se algum time ficar com menos jogadores, redistribuir os jogadores restantes
    let jogadoresSobrando = times.filter(time => time.length < numeroPessoasPorTime.value).flat();
    times.forEach(time => time.somaNiveis = time.reduce((acc, jogador) => acc + jogador.nivel, 0));
    
    for (let jogador of jogadoresSobrando) {
        let timeMenorNivel = times.reduce((menor, time) => time.somaNiveis < menor.somaNiveis ? time : menor, times[0]);
        timeMenorNivel.push(jogador);
        timeMenorNivel.somaNiveis += jogador.nivel;
    }

    emit('montarTimes', times.map(time => time));
};
</script>

<template>
    <div class="bold">Nome do jogador</div>
    <input v-model="novoJogador.nome" type="text" class="mt-10" placeholder="Digite o nome do jogador..." />

    <div class="mt-20 bold">Nível do jogador</div>
    <div id="conteudo-nivel-jogador">
        <div class="coluna" v-for="(nivel, index) in niveis" :key="nivel.id" @click="selecionarNivel(nivel.id)" :class="{'texto-roxo': novoJogador.nivel === nivel.id}">
            <div>{{ nivel.icone  }} {{ nivel.descricao }}</div>
        </div>
    </div>

    <div id="area-botao-secundario">
        <div v-if="estaAtualizando()" @click="removerJogador" class="mt-20">❌ remover jogador</div>
        <button @click="adicionarJogador" class="botao-secundario mt-20">{{ estaAtualizando() ? 'Atualizar jogador' : 'Adicionar jogador' }}</button>
    </div>

    <div class="bold mt-20">Número de pessoas por time</div>
    <input id="numero-pessoas-input" v-model="numeroPessoasPorTime" class="mt-10" type="number" placeholder="0" />

    <div class="bold texto-roxo mt-20">Jogadores</div>
    <div v-if="jogadores.length == 0" class="texto-cinza-claro mt-10">Ainda nenhum jogador foi selecionado</div>
    <div class="mt-10" v-for="(jogador, index) in jogadores">
        <div style="display: inline-block;" :class="{'bold': index == indexJogadorSendoAtualizado}" @click="selecionarJogador(jogador, index)">{{ obterIconePorNivel(jogador.nivel) }} {{ jogador.nome }}</div>
    </div>

    <div style="margin-bottom: 100px;"></div>

    <div id="area-botao-primario">
        <button @click="montarTimes" :class="{'botao-desativado': jogadores.length < 3 || numeroPessoasPorTime < 2, 'botao-primario': jogadores.length >= 3 && numeroPessoasPorTime >= 2}">Montar times</button>
    </div>
</template>

<style scoped>
#numero-pessoas-input {
    width: 20%;
}

input {
    width: 100%;
    padding: 0.8rem;
    border: 2px solid #DADADA;
    border-radius: 0.5rem;
    outline: none;
}

input:focus {
    border: 2px solid #865df6;
}

#conteudo-nivel-jogador {
    display: flex;
    flex-wrap: wrap ;
}

.coluna {
    width: 50%;
    padding: 0.5rem 0;
    box-sizing: border-box;
}

#area-botao-secundario {
    display: flex;
    justify-content: flex-end;
    align-items: center;
}

#area-botao-secundario > div {
    color: #ea0000;
    margin-left: auto;
}

.botao-secundario {
    margin-left: auto;
}

#area-botao-primario > button {
    position: fixed;
    bottom: 20px;
    width: 80vw;
    left: 50%;
    transform: translateX(-50%);
}
</style>