<template>
    <div v-if="info != null"> <!-- only renders page if data exists  -->
        <div class="card" style="width: 18rem;">
            <div class="card-body">
                <h5 class="card-title">
                    <i class="fas fa-dollar-sign"></i> 
                    Current Price: {{ info[currentFiatConversion] | toCurrency(currentFiatConversion)}}
                </h5>
                <h6 class="card-subtitle mb-2 text-muted">
                    <p v-bind:class="{ 'badge rounded-pill bg-success': priceUp, 'badge rounded-pill bg-danger': priceDown }">24h: {{info[currentFiatConversion+'_24h_change'] | toOneDecimal}}%</p>
                </h6>
                <p class="card-text">
                    <select v-model="currentFiatConversion">
                        <option value="aud">AUD</option>
                        <option value="cad">CAD</option>
                        <option value="eur">EUR</option>
                        <option value="gbp">GBP</option>
                        <option value="usd">USD</option>
                    </select>
                </p>
                <p class="card-text">
                    <button v-on:click="refresh" class="btn btn-outline-success">
                        <i class="fas fa-sync-alt"></i> Refresh
                    </button>
                </p>
            </div>
        </div>
    </div>
</template>
<script>
import axios from 'axios';
export default {
  name: 'CurrentPrice',
    data(){
        return {
            info: null,
            priceUp: false,
            priceDown: false,
            token: null,
            currentFiatConversion: 'aud',
            percentChange: null
        }
    },
    props: {
        ct: null,
        tokenData:null
    },
    mounted(){
        if (localStorage.currentCurrency) {
            this.currentFiatConversion = localStorage.currentCurrency;
        }
        this.refresh(); 
    },
    watch:{
        ct(){
            this.refresh();
        },
        currentFiatConversion(){
            localStorage.currentCurrency = this.currentFiatConversion;
            this.refresh();
        },
        tokenData(){
            this.updateProgressBar();
        }
    },
    methods:{
        refresh(){
            if(this.ct != null){
                axios
                .get(`https://api.coingecko.com/api/v3/simple/token_price/${this.ct.asset_platform_id}?contract_addresses=${this.ct.contract_address}&vs_currencies=${this.currentFiatConversion}&include_24hr_change=true`,{
                headers:{
                    'accept':'application/json' 
                }
                }).then(response => {
                    if(JSON.stringify(this.info) === JSON.stringify(response.data[this.ct.contract_address])){
                        console.log('Data not refreshed, data has not changed.')
                    }else{
                        this.info = response.data[this.ct.contract_address];
                        //console.log(this.info);
                        this.priceChangeToggle();
                        this.updateProgressBar();
                        this.$emit('cp', this.info[this.currentFiatConversion]);
                        this.$emit('cc',this.currentFiatConversion);
                    }   
                })  
            }
        },

        priceChangeToggle(){
            if(this.info != null){
                if(this.info[this.currentFiatConversion+'_24h_change'] > 0){
                    this.priceUp = true;
                    this.priceDown = false;
                }else{
                    this.priceDown = true;
                    this.priceUp = false;

                }
            }
        },

        updateProgressBar(){
            if(this.tokenData){
                let ath = this.tokenData.market_data.ath[this.currentFiatConversion];
                let atl = this.tokenData.market_data.atl[this.currentFiatConversion];

                this.percentChange = (((ath-atl)/0.5) * 100) + '%';
                //this.percentChange = 5 + '%';
            }
        }

    },
    filters:{
        toCurrency (value,currency) {
            if (typeof value !== "number") {
                return value;
            }
            var formatter = new Intl.NumberFormat('en-US', {
                style: 'currency',
                currency: currency,
                minimumFractionDigits: 8
            });
            return formatter.format(value);
        },

        toOneDecimal(value){
            if (typeof value !== "number") {
                return value;
            }
            return value.toFixed(1);
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.badge{
    color: white !important;
}
</style>