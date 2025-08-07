<script setup>
import { computed, onMounted, ref, watch } from 'vue'

const filmes = ref([])

const adicionandoFilme = ref(false)

const nomeFilme = ref(null)
const imagemFilme = ref(null)
const lancamentoFilme = ref(null)
const generoFilme = ref(null)

function salvarFilme() {
  if (
    nomeFilme.value != null &&
    imagemFilme.value != null &&
    lancamentoFilme.value != null &&
    generoFilme.value != null
  ) {
    filmes.value.push({
      nome: nomeFilme.value,
      lancamento: lancamentoFilme.value,
      genero: generoFilme.value,
      imagem: imagemFilme.value
    })
    limpaEFecha()
  } else {
    alert('Informações inclompletas')
  }
}

function limpaEFecha() {
  nomeFilme.value = null
  lancamentoFilme.value = null
  generoFilme.value = null
  imagemFilme.value = null
  adicionandoFilme.value = false
}

function excluirFilme(i) {
  if (confirm(`Tem certeza que deseja excluir o filme: ${filmes.value[i].nome} ?`)) {
    filmes.value.pop(filmes.value[i])
  }
}

function definirLike(i, like) {
  if (filmes.value[i].like === true && like === true) {
    filmes.value[i].like = null
  } else if (filmes.value[i].like === false && like === false) {
    filmes.value[i].like = null
  } else {
    filmes.value[i].like = like
  }
}

//todos | gostei | naoGostei

const filtro = ref('todos')

function definirFilmes() {
  switch (filtro.value) {
    case 'gostei':
      return filmes.value.filter((item) => item.like === true)
    case 'naoGostei':
      return filmes.value.filter((item) => item.like === false)
    default:
      return filmes.value
  }
}

const filmesParaExibir = computed(definirFilmes)

function persistencia(novoValor, antigoValor) {
  localStorage.setItem('vueflix', JSON.stringify(novoValor))
}
watch(filmes, persistencia, { deep: true })

function ahPersistencia() {
  const dadosLocalStorange = localStorage.getItem('vueflix')
  if(dadosLocalStorange){
    filmes.value = JSON.parse(dadosLocalStorange)
  }
}

onMounted(ahPersistencia)
</script>

<template>
  <div class="vueflix">
    <div class="acoes-usuario">
      <div class="filtros">
        <div class="titulo">Filtrar</div>
        <div class="opcoes-filtros">
          <button @click="filtro = 'todos'" class="botao" :class="{ ativo: filtro === 'todos' }">
            Todos
          </button>
          <button @click="filtro = 'gostei'" class="botao" :class="{ ativo: filtro === 'gostei' }">
            Gostei
          </button>
          <button
            @click="filtro = 'naoGostei'"
            class="botao"
            :class="{ ativo: filtro === 'naoGostei' }"
          >
            Não Gostei
          </button>
        </div>
      </div>

      <div class="novo-filme">
        <div v-if="adicionandoFilme" class="adicionar-filme">
          <input v-model="nomeFilme" type="text" autocomplete="off" placeholder="Nome do Filme" />
          <input v-model="imagemFilme" type="text" autocomplete="off" placeholder="URL da Imagem" />
          <input
            v-model="lancamentoFilme"
            type="text"
            autocomplete="off"
            placeholder="Lançamento"
          />
          <input v-model="generoFilme" type="text" autocomplete="off" placeholder="Gênero" />
          <div class="acoes">
            <button class="botao ativo" @click="salvarFilme">Salvar</button>
            <button class="botao danger ativo" @click="limpaEFecha">Cancelar</button>
          </div>
        </div>
        <button v-else class="botao ativo" @click="adicionandoFilme = true">Adicionar Filme</button>
      </div>
    </div>

    <div class="filmes">
      <div v-for="(filme, i) in filmesParaExibir" class="filme">
        <div class="capa-container">
          <div class="acoes-filme">
            <button
              @click="definirLike(i, true)"
              class="botao"
              :class="{ ativo: filme.like === true }"
            >
              Gostei
            </button>
            <button
              @click="definirLike(i, false)"
              class="botao danger"
              :class="{ ativo: filme.like === false }"
            >
              Não Gostei
            </button>
            <button @click="excluirFilme(i)" class="botao danger">Excluir</button>
          </div>
          <img class="capa" :src="filme.imagem" alt="" />
        </div>
        <div class="nome">{{ filme.nome }}</div>
        <div class="info">{{ filme.lancamento }} - {{ filme.genero }}</div>
      </div>
      <p
        v-if="filmesParaExibir.length === 0"
        :style="{ flex: 1, textAlign: 'center', marginTop: '16px' }"
      >
        Não há filmes para exibir.
      </p>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.vueflix {
  padding: 16px;

  .acoes-usuario {
    display: flex;
    justify-content: space-between;
    align-items: end;
    margin: 30px 0;

    .adicionar-filme {
      display: flex;
    }
  }

  .filmes {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
    min-height: 350px;

    .filme {
      .capa-container {
        width: 200px;
        height: auto;
        position: relative;
        .capa {
          width: 100%;
          height: 100%;
        }

        .acoes-filme {
          position: absolute;
          bottom: 0;
          padding-bottom: 12px;
          background-image: linear-gradient(to bottom, rgba(255, 0, 0, 0), rgb(0, 0, 0));
          display: none;
          flex-direction: column;
          width: 100%;
          height: 100%;
          justify-content: flex-end;
        }
      }

      .nome {
        font-weight: bold;
      }

      .info {
        font-size: 12px;
      }

      &:hover {
        .acoes-filme {
          display: flex;
        }
      }
    }
  }
}
</style>
