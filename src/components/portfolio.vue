<template>
  <div>
    <div class="card" style="width: 18rem;">
      <div class="card-body">
        <h5 class="card-title" v-if="prices != null"><i class="fas fa-money-bill-alt"></i> Portfolio Value: {{totalFolioValue | toCurrency(currency)}}</h5>
        <h6 v-if="nonNullTokens != null" class="card-subtitle mb-2 text-muted">Tokens included in value</h6>
        <p class="card-text" v-if="nonNullTokens != null"> 
            <ul id="tokenList">
                <li v-for="(token,key) in nonNullTokens" :key=key><span class="badge badge-dark ml-1">{{token}}</span></li>
            </ul>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import web3 from 'web3';
export default {
  name: 'portfolio',
  data(){
    return {
      calculatedAmount: 0,
      tokenData: null,
      prices: null,
      totalFolioValue: null,
      tokenBalances: null,
      acct_ID: null,
      nonNullTokens: null,
      tokenABI: [{"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_from","type":"address"},{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transferFrom","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"payable":true,"stateMutability":"payable","type":"fallback"},{"anonymous":false,"inputs":[{"indexed":true,"name":"owner","type":"address"},{"indexed":true,"name":"spender","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Approval","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"from","type":"address"},{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Transfer","type":"event"}],
    }
  }, 
  props: {
    tokens: null,
    currency: null,
    watchUpdate: null
  },
  watch:{
      tokenData(){ 
        this.getPrices();
      },
      currency(){
          console.log(this.currency)
          this.getPrices();
      },
      watchUpdate(){
          this.getPrices();
          console.log("Folio Notified!");
      },
      tokenBalances(){
          this.calculate();
      }
  },   
  mounted(){
    this.getMetaData();
    if (localStorage.acct_ID) {
        this.acct_ID = localStorage.acct_ID;
        console.log(this.acct_ID)
        this.getBalances();
    }
  },
  methods:{ 
      
    getMetaData(){
      let promiseArray = [];
      for(const token in this.tokens){
        promiseArray.push(axios.get(`https://api.coingecko.com/api/v3/coins/${token}`,{
        headers:{
            'accept':'application/json' 
        }
        }))
      }
      const results = Promise.all(promiseArray).catch(err=>console.log(err))
      results.then(data =>{
        let dataArray = []; 
        data.forEach(p => {
            dataArray.push(p.data);   
        });
        //console.log(dataArray);
        this.tokenData = dataArray;
        this.getPrices();
      })
      
    },
    getPrices(){
        
        let promiseArray = [];
        if (this.acct_ID == null && localStorage.acct_ID) { // this is done so that acct_id can be set here if it becomes available in local storage after mounted()
            this.acct_ID = localStorage.acct_ID;
        }
        if (this.tokenData != null){
           for(let i = 0; i < this.tokenData.length; i++){
            promiseArray.push(axios.get(`https://api.coingecko.com/api/v3/simple/token_price/${this.tokenData[i].asset_platform_id}?contract_addresses=${this.tokenData[i].contract_address}&vs_currencies=${this.currency}`,{
            headers:{
                'accept':'application/json' 
                }
                }))
                
            }
            const results = Promise.all(promiseArray).catch(err=>console.log(err))
            results.then(data =>{
                let dataArray = []; 
                data.forEach(p => {
                    dataArray.push(p.data);   
                });
                //console.log(dataArray);
                this.prices = dataArray;
                this.getBalances();
            })
        }else{
            this.getMetaData();
        }
    },
    getBalances(){
        if(this.acct_ID != null && this.tokenData != null){
                //Web3 get token balance dynamically
                const balancesArray = [];  
                const nonNullBalances = [];
                const web3Obj = new web3('https://bsc-dataseed1.binance.org'); //step 1

                for(let i=0; i<this.tokenData.length; i++){
                        var tokenInst = new web3Obj.eth.Contract(this.tokenABI,this.tokenData[i].contract_address); //step 2
                        tokenInst.methods.balanceOf(this.acct_ID,).call().then((res)=>{ //step 3
                        var bal = web3.utils.fromWei(res,'nano');
                        bal = (Math.round(bal * 100) / 100).toFixed(3);
                        balancesArray.push(bal)
                        //console.log(balancesArray);
                        if(bal != 0){
                            nonNullBalances.push(this.tokenData[i].name)
                        }
                    });
                }

                this.tokenBalances = balancesArray
                this.nonNullTokens = nonNullBalances
                this.calculate();
            }  
    },

    calculate(){
        if(this.tokenBalances != null && this.prices != null && this.tokenData != null){

            let total = 0;
            for(let i = 0; i < this.tokenBalances.length; i++){
                total += (this.prices[i][this.tokenData[i].contract_address][this.currency] * this.tokenBalances[i]); 
            }
            console.log(total);
            this.totalFolioValue = total;
        }
    }
  },
  filters:{
        toCurrency (value, currency) {
            if (typeof value !== "number") {
                return value;
            }
            var formatter = new Intl.NumberFormat('en-US', {
                style: 'currency',
                currency: currency,
                minimumFractionDigits: 3
            });
            return formatter.format(value);
        }
    }
}
</script>

<style>
    #tokenList li{
        display: inline;
        
    }
    ul#tokenList{
       padding-inline-start: 0px; 
    }

    .mr{
        margin-right: 20px ;
    }
</style>
