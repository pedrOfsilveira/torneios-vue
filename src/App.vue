<script setup>
import { ref } from 'vue';

const newNome = ref('');
const jogadores = ref([]);
const partidas = ref([]);
const partidasFinalizadas = ref([]);
const vencedores = ref([]);
const on = ref(false);

class Jogador {
  constructor(nome) {
    this.nome = nome;
    this.pontos = 0;
  }
}

const addJogador = () => {
  if (newNome.value !== '') {
    const jogador = ref(new Jogador(newNome.value));
    jogadores.value.push(jogador.value);
    newNome.value = '';
  } else {
    alert('Preencha o nome do jogador!');
  }
};

const deleteJogador = index => {
  jogadores.value.splice(index, 1);
};

const geraPartidas = competidores => {
  console.log(competidores);
  partidas.value = competidores.flatMap((competidor, i) =>
    competidores
      .slice(i + 1)
      .map(outroCompetidor => [competidor, outroCompetidor])
  );
  console.log(partidas);
};

const start = () => {
  geraPartidas(jogadores.value);

  localStorage.setItem('jogadores', JSON.stringify(jogadores.value));
  localStorage.setItem('partidas', JSON.stringify(partidas.value));

  on.value = true;
};

const load = () => {
  const storedJogadores = localStorage.getItem('jogadores');
  const storedPartidas = localStorage.getItem('partidas');
  const storedPartidasFinalizadas = localStorage.getItem('partidasFinalizadas');
  const storedVencedores = localStorage.getItem('vencedores');

  if (storedJogadores && storedPartidas) {
    partidas.value = JSON.parse(storedPartidas);
    partidasFinalizadas.value = JSON.parse(storedPartidasFinalizadas);
    vencedores.value = JSON.parse(storedVencedores);
    jogadores.value = JSON.parse(storedJogadores);

    on.value = true;
  }
};

const registraVitoria = (index, vencedorIndex) => {
  partidas.value[index][vencedorIndex].pontos++;
  console.log(partidas.value[index][vencedorIndex].pontos);

  partidasFinalizadas.value[index] = true;
  vencedores.value[index] = vencedorIndex;
  console.log(vencedores.value);

  localStorage.setItem('partidasFinalizadas', JSON.stringify(partidasFinalizadas.value));
  localStorage.setItem('jogadores', JSON.stringify(jogadores.value));
  localStorage.setItem('vencedores', JSON.stringify(vencedores.value));
};
</script>

<template>
  <main class="wrapper">
    <aside>
      <div class="titulo-aside">
        <img src="./assets/icons/peao.png" alt="peão de xadrez" />
        <h3>Como usar?</h3>
      </div>
      <hr />
      <ol>
        <li>Preencha o nome do jogador</li>
        <li>
          Quando estiver tudo pronto, aperte o botão
          <span class="verde">Começar</span>
        </li>
        <li>
          Marque o resultado dos jogos, de acordo com o decorrer do torneio
        </li>
        <li>Aproveite o torneio!</li>
      </ol>
    </aside>
    <div class="container">
      <div class="titulo">
        <img src="./assets/icons/torneio.svg" alt="uma medalha de ouro" />
        <h1>Torneios</h1>
      </div>

      <div class="caixa" id="caixa-nomes" v-if="!on">
        <div class="titulo-linha">
          <h3 class="titulo-caixa">Adicionando jogadores</h3>
          <hr />
        </div>
        <div class="add-nome">
          <form>
            <input class="nome" autocomplete="off" id="nome" placeholder="Nome" type="text" v-model="newNome" />

            <button class="button add" id="add" type="submit" @click.prevent="addJogador">
              Adicionar
            </button>
          </form>
        </div>

        <table id="tabela-jogadores">
          <thead>
            <tr>
              <th>Jogadores</th>
            </tr>
          </thead>
          <tbody id="tbody-jogadores">
            <tr id="vazio" v-if="!jogadores.length">
              <td>Vazio por agora.</td>
            </tr>
            <tr v-for="(jogador, index) in jogadores" :key="jogador">
              <td>{{ jogador.nome }}</td>
              <td class="td-botao">
                <button class="delete" @click="deleteJogador(index)">
                  <img src="./assets/icons/x.png" alt="x">
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div id="caixa-partidas" class="caixa" v-if="on">
        <div class="titulo-linha">
          <h3 class="titulo-caixa">Partidas</h3>
          <hr />
        </div>
        <div id="partidas">
          <div class="partida" v-for="(partida, index) in partidas" :key="partida">
            <div class="partida-nomes">
              <p class="nome1" :class="{
                vermelho: vencedores[index] === 1,
                verde: vencedores[index] === 0,
              }">{{ partida[0].nome }}</p>
              <p class="nome2" :class="{
                vermelho: vencedores[index] === 0,
                verde: vencedores[index] === 1,
              }">{{ partida[1].nome }}</p>
            </div>
            <div class="partida-botoes">
              <button class="vencer" :class="{ perdedor: vencedores[index] === 1 }"
                :disabled="partidasFinalizadas[index]" @click="registraVitoria(index, 0)">
                <img src="./assets/icons/vencer.png" alt="V">
              </button>
              <button class="vencer" :class="{ perdedor: vencedores[index] === 0 }"
                :disabled="partidasFinalizadas[index]" @click="registraVitoria(index, 1)">
                <img src="./assets/icons/vencer.png" alt="V"> 
              </button>
            </div>
            <hr />
          </div>
        </div>
        <table id="tabela-ranking">
          <thead>
            <tr>
              <th>Ranking</th>
            </tr>
          </thead>
          <tbody id="tbody-ranking">
            <tr v-for="(jogador, index) in jogadores.sort(
              (a, b) => b.pontos - a.pontos
            )" :class="{
              ouro: index === 0,
              prata: index === 1,
              bronze: index === 2,
            }">
              <td>
                #{{ index + 1 }} {{ jogador.nome }}
              </td>
              <td class="pontos">{{ jogador.pontos }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="botoes" v-if="!on">
        <button class="start button" id="start" @click="start">Começar</button>
        <button class="load button preto" id="load" @click="load">
          Carregar
        </button>
      </div>
    </div>
  </main>
</template>
