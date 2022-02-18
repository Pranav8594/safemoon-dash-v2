<template>
  <div id="app" class="card-columns">
    <CurrentPrice @cp="currentPrice" :ct="tokenMeta" @cc="curretSelectedCurrency" :tokenData="tokenMeta"/>
    <CurrentValue @ct="getMetaData" :tokens="tokens" :tokenMeta="tokenMeta" :currency="currency" :currentPrice="cpd" :currentBalance="cbd"/>
    <Portfolio :tokens="tokens" :currency="currency" :watchUpdate="notifyFolio"/>
    <FutureGrowth :currentTokenBalance="cbd" :token="tokenMeta" :toCurrency="currency" />
    <ConnectWallet @cb="currentBalance" :tokenInfo="tokenMeta"/>
    <Donations/>
  </div>
</template>

<script>
import CurrentPrice from './components/CurrentPrice.vue'
import CurrentValue from './components/currentValue.vue'
import ConnectWallet from './components/connectWallet.vue'
import FutureGrowth from './components/futureGrowth.vue'
import Donations from './components/donations.vue'
import Portfolio from './components/portfolio'

import axios from 'axios';
export default {
  name: 'App',
  components: {
    CurrentPrice,
    CurrentValue,
    ConnectWallet,
    FutureGrowth,
    Donations,
    Portfolio
  },
  data(){
    return {
      cpd: null,
      cbd: null,
      currentToken: 'safemoon-2',
      tokenMeta: null,
      currency: 'aud',
      tokens: {"safemoon-2":"Safemoon","bonfire":"Bonfire","elongate": "Elongate", "eclipse-2":"Eclipse","useless":"Useless"},
      notifyFolio: false
    }
  }, 
  mounted(){
    this.getMetaData(this.currentToken);
  },
  methods:{
    currentPrice(value){
      this.cpd = value;
      this.notifyFolio = !this.notifyFolio;
    },
    currentBalance(value){
      this.cbd= value;
      this.notifyFolio = !this.notifyFolio;
    },
    curretSelectedCurrency(value){
      this.currency = value;
    },
    getMetaData(token){
      axios.get(`https://api.coingecko.com/api/v3/coins/${token}`,{
        headers:{
            'accept':'application/json' 
        }
        }).then(response => {
          this.tokenMeta = response.data
          console.log(response.data)
        });
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 30px;
  
}
.rounded{
  border-radius: 100%;
}
/* .card{
  background-color: #12181b;
  color: white;
} 

/* input[type="text"], textarea {

  background-color : #12181b; 

} */
/* Float four columns side by side */
.column {
  float: left;
  width: 25%;
  padding: 0 10px;
}
/* Remove extra left and right margins, due to padding in columns */
.row {margin: 0 -5px;}
/* Clear floats after the columns */
.row:after {
  content: "";
  display: table;
  clear: both;
}
/* Responsive columns - one column layout (vertical) on small screens */
@media (min-width: 576px) {
    .card-columns {
        column-count: 2;
    }
}

@media (min-width: 864px) {
    .card-columns {
        column-count: 2;
    }
}

@media (min-width: 1024px) {
    .card-columns {
        column-count: 3;
    }
}

@media (min-width: 1200px) {
    .card-columns {
        column-count: 4;
    }
}
@media (min-width: 1500px) {
    .card-columns {
        column-count: 5;
    }
}
</style>
