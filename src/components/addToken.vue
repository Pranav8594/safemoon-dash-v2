<template>
  <div v-if="info != null">
    <!-- only renders page if data exists  -->
    <div class="card" style="width: 18rem">
      <div class="card-body">
        <h5 class="card-title"><i class="fas fa-dollar-sign"></i></h5>
        <h6 class="card-subtitle mb-2 text-muted"></h6>
        <p class="card-text">
          <button v-on:click="add" class="btn btn-outline-success">
            <i class="fas fa-sync-alt"></i> Add
          </button>
        </p>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
export default {
  name: "CurrentPrice",
  data() {
    return {
      token: null,
    };
  },
  mounted() {
      
  },
  watch: {},
  methods: {
    add() {
      axios
        .get(
          `https://api.coingecko.com/api/v3/simple/token_price/${this.ct.asset_platform_id}?contract_addresses=${this.ct.contract_address}&vs_currencies=${this.currentFiatConversion}&include_24hr_change=true`,
          {
            headers: {
              accept: "application/json",
            },
          }
        )
        .then((response) => {
          
        });
    },
  },
  filters: {
   
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.badge {
  color: white !important;
}
</style>