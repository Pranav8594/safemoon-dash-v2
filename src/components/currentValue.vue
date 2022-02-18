<template>
    <div>
        <div class="card" style="width: 18rem;">
            <div class="card-body">
                <h5 class="card-title"><i class="fas fa-piggy-bank"></i> Current Value: {{currentPrice*currentBalance | toCurrency(currency) }} 
                    <i v-bind:class="{'red': !toggleCaret }" v-if="!toggleCaret" class="fas fa-caret-down"></i>
                    <i v-bind:class="{ 'green': toggleCaret }" v-if="toggleCaret" class="fas fa-caret-up"></i>
                </h5>
                <h6 v-if="tokenMeta != null" class="card-subtitle mb-2 text-muted rounded">
                    <img v-bind:src="tokenMeta.image.small" />
                </h6>
                <p class="card-text">
                  <span v-if="tokenMeta != null" class="badge badge-pill badge-info" >ATH: {{tokenMeta.market_data.ath[currency] * currentBalance | toCurrency(currency)}} </span>  
                </p>
                <p class="card-text">
                <span>
                    <select v-model="currentToken">
                        <option v-for="(token,key) in tokens" :key=key :value="key">
                            {{token}}
                        </option>
                    </select>
                </span>
                </p>
            </div>
        </div>
    </div>
</template>

<script>
export default {
  name: 'CurrentValue',
  data(){
    return {
      currentToken: 'safemoon-2',
      toggleCaret: false
    }
  }, 
  props:{
     currentPrice: null,
     currentBalance: null,
     tokens: null,
     tokenMeta: null,
     currency: null 
  },
  watch:{
    currentToken(){
      this.$emit('ct', this.currentToken);
    }
  }, 
  mounted(){
        
        this.setCaret(); 
  },
  methods:{
      setCaret(){
          let cv = this.currentPrice * this.currentBalance;
          let ath = this.tokenMeta.market_data.ath[this.currency] * this.currentBalance;

          if(cv > ath){
              this.toggleCaret = true;
          }else{
              this.toggleCaret = false;
          }
          console.log(this.toggleCaret)
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
<style scoped>
.red{
    color: #dc3545;
}
.green{
    color: #28a745;
}
</style>