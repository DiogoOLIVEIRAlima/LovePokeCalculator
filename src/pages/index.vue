<script setup>
import { ref, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const router = useRouter()
const route = useRoute()

let cardName1 = ref("")
let cardName2 = ref("")
let imagem1 = ref("")
let imagem2 = ref("")
let input1 = ref("")
let input2 = ref("")
let showVAlert = ref(false)
let textAlert = ref("")
let showResult = ref(false)
let porcentagem = ref("")
let texto = ref("")

const baseUrl = 'https://api.pokemontcg.io/v2/cards'
const options = {
  headers: {
    'x-api-key': "1cc070af-abbf-4b86-b2ad-09bcdbd6a540"
  }
};

const loveBaseUrl = "https://love-calculator.p.rapidapi.com/getPercentage"
const loveOption = {
  headers: {
    'X-RapidAPI-Key': 'e7772c2724msh7840225d36ed3e6p14712cjsnfaddae21c8f2',
    'X-RapidAPI-Host': 'love-calculator.p.rapidapi.com'
  }
}

function fetchData(firstCard) {
  let params = {
    q : `!name:"${firstCard == true ? input1.value : input2.value}"`,
    pageSize: 10 
  }

  let url = new URL(baseUrl);
  url.search = new URLSearchParams(params).toString();

  fetch(url, options)
    .then(response => {
      if(response.ok){
        return response.json()
      }

      throw new Error(`Falha na solicitação: ${response.status}`)
    })
    .then(data => {
      let dados = data.data[0]
      if(firstCard == true){
        
        cardName1 = dados.name
        imagem1.value = dados.images.small
        input1.value = ""
      }
      else{
        cardName2 = dados.name
        imagem2.value = dados.images.small
        input2.value = ""
      }
    })
    .catch(error => {
     showError("Infelizmente não achamos esse pokemon 😞")
    })

}


function showError(texto) {
  showVAlert.value = true
  textAlert.value = texto

  setTimeout(() =>{
    showVAlert.value = false
    textAlert.value = ""
  }, 3000)
}


function calcular() {
  if(cardName1 === cardName2){
    showError("Ei, escolha dois pokemons diferentes. Isso seria meio estranho 🤢")
    return
  }

  if(cardName1.value == "" || cardName2.value == ""){
    showError("Digite o nome de dois pokemons para ver o percentual de amor deles 😏")
    return
  }

  let paramsLove = {
    sname: cardName1,
    fname: cardName2 
  }

  let urlLove = new URL(loveBaseUrl);
  urlLove.search = new URLSearchParams(paramsLove).toString();

  fetch(urlLove, loveOption)
    .then(response => {
      if(response.ok){
        return response.json()
      }

      throw new Error(`Erro na response: ${response.status}`)
    })
    .then( data => {

      porcentagem.value = `${data.percentage}%`
      texto.value = data.result

      showResult.value = true
    })
    .catch(error => {
      showError("Opps algo de errado não deu certo")
    })
}

</script>


<template>
 <h1>Seja Bem-Vindo ao PokeLove Calculator ❤️</h1>
  <VRow>
    <VCol cols="12">        
        <VRow align="center" style="padding-left: 200px; padding-right: 200px;">
          <!-- Parte esquerda do card -->
          <VCol lg="4" sm="4" md="4" cols="4" class="cardNome">
            <div class="container-img">
              <img src="../assets/cardPokemon.jpg" v-if="cardName1 == '' ">
              <img :src="imagem1" alt="" v-else>
            </div>
            <p>{{ cardName1 }}</p>
            <VTextField
              label="Digite o nome da carta"
              class="custom-gradient-border input"
              autofocus
              v-model="input1"
              @keydown.enter="fetchData(true)"      
              />
            <VBtn @click="fetchData(true)" color="green" width="250">
              Buscar
            </VBtn>
          </VCol>

          <!-- Parte central do card -->
          <VCol lg="4" sm="4" md="4" cols="4" class="cardButton">
            <p v-if="showResult" id="porcentagem">{{ porcentagem }}</p>
            <p v-if="showResult" id="texto">{{ texto }}</p>
            <VBtn color="red" dark @click="calcular">Calcular 💗</VBtn>
          </VCol>

          <!-- Parte direita do card -->
          <VCol lg="4" sm="4" md="4" cols="4" class="cardNome">
            <div class="container-img">
              <img src="../assets/cardPokemon.jpg" v-if="cardName2 == ''">
              <img :src="imagem2" alt="" v-else >
            </div>
            <p>{{cardName2}}</p>
            <VTextField
              label="Digite o nome da carta"
              class="custom-gradient-border input"
              autofocus
              v-model="input2"
              @keydown.enter="fetchData(false)"
              />
            <VBtn @click="fetchData(false)" color="green" width="250">
              Buscar
            </VBtn>
          </VCol>
        </VRow>
    </VCol>
  </VRow>

  <VAlert :model-value="showVAlert" title="Erro" :text="textAlert" type="error" class="alert" position="absolute"
      :max-width="700" :max-height="100" />
</template>

<style>

.alert {
  position: absolute;
  right: 0;
  top: 15px;
  z-index: 100000000;
}

h1 {
  text-align: center;
  margin: 20px;
}

.v-main {
  background-color: black !important;
  color: white !important;
}

.v-card {
  margin: auto;
  padding: 2rem !important;
}


.container-img {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 250px;
  height: 300px;
}

img {
  width: 100%;
  height: 100%;
}

.cardNome {
  display: flex !important;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.cardButton {
  display: flex !important;
  justify-content: center;
  flex-direction: column;

  align-items: center;
}

.input {
  width: 250px !important;
}

#porcentagem {
  font-family: fantasy;
  font-size: 64px;
  font-weight: bold;
  margin-top: -50%;
}

#texto {
  font-size: 12px;
  margin-bottom: 10%;
  
}


</style>
