<template>
    <div>
        <div class="card" style="width: 18rem;">
            <div class="card-body">
                <h5 class="card-title">
                    <i class="fas fa-rocket"></i> 
                    Future Value: {{futureValue*currentTokenBalance | toCurrency(toCurrency)}}
                </h5>
                <h6 v-if="token != null" class="card-subtitle mb-2 text-muted">{{token.name}}</h6>
                <div class="card-text">
                    <div class="row">
                        <div class="col">
                            <input v-model="futureValue" type="text" class="form-control" placeholder="Future Price">
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
export default {
  name: 'futureGrowth',
    data(){
        return {
            tokenBal: null,
            futureValue: null
        }
    },
    props: {
        currentTokenBalance: null,
        toCurrency: null,
        token: null,
        
    },
    filters:{
        toCurrency (value,currency) {
            if (typeof value !== "number" && currency != null) {
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

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>