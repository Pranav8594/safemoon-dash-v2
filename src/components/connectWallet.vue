<template>
  <div>
    <div class="card" style="width: 18rem;">
        <div class="card-body">
            <h5 class="card-title">Wallet <i class="fas fa-wallet"></i></h5>
            <h6 class="card-subtitle mb-2 text-muted"><p v-if="acct_ID != null">{{acct_ID.slice(0,8)+'...'}}</p></h6>
            <!-- <p class="card-text"><button class="btn btn-outline-info" :disabled="btnDisabled" v-on:click="connect">{{btnTitle}}</button></p> -->
            <input type="text" class="form-control" v-model="acct_ID" placeholder="Your Wallet's Public Address">
            <p class="card-text mt-3"><button class="btn btn-outline-success" v-on:click="detectTokenBalance()"><i class="fas fa-coins"></i> Get Balance</button></p>
            <p class="card-text" v-if="tokenBalance != null"><span class="badge bg-primary">{{tokenInfo.name}}:{{tokenBalance}}</span></p>
            <p class="red">{{msg}}</p>
        </div>
    </div>
  </div>

</template>

<script>

//import axios from 'axios';
import web3 from 'web3';
export default { 
  name: 'ConnectWallet',
  data(){
      return {
            acct_ID: null,
            tokenABI: [{"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_from","type":"address"},{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transferFrom","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"}],"name":"balanceOf","outputs":[{"name":"balance","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"_owner","type":"address"},{"name":"_spender","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"payable":true,"stateMutability":"payable","type":"fallback"},{"anonymous":false,"inputs":[{"indexed":true,"name":"owner","type":"address"},{"indexed":true,"name":"spender","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Approval","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"name":"from","type":"address"},{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Transfer","type":"event"}],
            tokenBalance: null,
            btnDisabled: false,
            msg: '',
            btnTitle: 'Connect Wallet'
        }
    },
    props:{
        tokenInfo: null
    },
    watch:{
        tokenInfo(){
            this.detectTokenBalance();
        }
    },
    mounted(){
        if (localStorage.acct_ID) {
            this.acct_ID = localStorage.acct_ID;
            this.detectTokenBalance();
        }
    },
  methods:{
      /*async connect(){
          const {ethereum} = window;
          if(ethereum && ethereum.isMetaMask){
              console.log('MetaMask is installed!');

               try {
                    // Will open the MetaMask UI
                    this.btnDisabled = true;
                    await ethereum.request({ method: 'eth_requestAccounts' }).then((value) =>{ 
                        this.acct_ID = value[0]
                        this.msg = '';
                        this.btnTitle = 'Wallet Connected!'
                        this.detectTokenBalance();
                    })
                } catch (error) {
                    console.error(error);
                    this.btnDisabled = false;
                    this.btnTitle = 'Connect Wallet'
                }
            }
        },*/
        detectTokenBalance () {
            if(this.acct_ID != null && this.tokenInfo != null){
                this.msg = '';
                //this.$emit('cb', 0); // set it to 0 fisrt everytime function runs
                localStorage.acct_ID = this.acct_ID;
                //Web3 get token balance dynamically  
                const web3Obj = new web3('https://bsc-dataseed1.binance.org'); //step 1
                var tokenInst = new web3Obj.eth.Contract(this.tokenABI,this.tokenInfo.contract_address); //step 2
                tokenInst.methods.balanceOf(this.acct_ID,).call().then((res,err)=>{ //step 3
                    var bal = web3.utils.fromWei(res,'nano');
                    bal = (Math.round(bal * 100) / 100).toFixed(3);
                    this.tokenBalance = bal.toString().replace(/\B(?<!\.\d*)(?=(\d{3})+(?!\d))/g, ",");
                    this.$emit('cb', bal);
                    if(err){
                        this.msg = err;
                    }
                });
            }else{
                this.msg = 'Please enter your Metamask or Trustwallet BSC address'
            }
            this.$gtag.event('get_bal_click', {
                'event_category': 'action',
                'event_label': 'Get balance button clicked!',
                'value': 1
            })   
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .red{
        color: red;
    }
    .badge{
        color: white;
    }
</style>