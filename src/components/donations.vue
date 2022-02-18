<template>
    <div>
        <div class="card" style="width: 18rem;">
            <div class="card-body">
                <h5 class="card-title"><i style="color:red" class="fas fa-heart"></i> Donate <i class="fas fa-hand-holding-usd"></i></h5>
                <h6 class="card-text mt-3">
                    <div class="input-group mb-3">
                        <input type="text" class="form-control" v-model="text" placeholder="Wallet Address" aria-describedby="button-addon2" readonly>
                        <button class="btn btn-outline-secondary"  v-on:click="copyAddress"  type="button" id="button-addon2">
                            <i class="fas fa-copy"></i> 
                            {{btnTitle}}
                        </button>
                    </div>
                </h6>
                <div class="card-text">
                    <select v-model="address">
                        <option value="">Select Coin</option>
                        <option value="bnb">BNB</option>
                        <option value="eth">ETH</option>
                        <option value="btc">BTC</option>
                        <option value="ada">ADA</option>
                    </select>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import copy from 'copy-text-to-clipboard';
export default {
  name: 'donations',

    data(){
        return {
            text: null,
            address: '',
            btnTitle: 'Copy'
        }
    },
    props: {
        
        
    },
    watch:{
        address(){
            this.displayAddress();
        }
        
    },
    mounted(){

    },
    methods:{
        
        displayAddress(){
            
            switch(this.address){
                case 'bnb':
                    this.text = '0x2e71a73714fF583aCD79463Bb062fd063d429B73';
                    break;
                case 'eth':
                    this.text = '0x774a63700C1E9e1e60Ba52651Ba8c2AD51D68854';
                    break;
                case 'btc':
                    this.text = 'bc1q0uyqdf9t38dt088fdptzd8arfhakmzmquad72h';
                    break;
                case 'ada':
                    this.text = 'addr1qxf0p20r4s77ujc4y6tst0x6yrfjwpwnl3pk69gefvh6feyj7z578tpaae932f5hqk7d5gxnyuza8lzrd523jje05njq7t9nny';
                    break;
                default:
                    this.text = '';
            }
        },

        copyAddress(){
            if(this.text != null){
                copy(this.text);
                this.btnTitle = 'Copied!';
                setTimeout(() => this.changeBtnTitle('Copy'),1000);
            }
            this.$gtag.event('copy_addr_click', {
                'event_category': 'action',
                'event_label': 'Donate Copy button clicked!',
                'value': 1
            }) 
        },

        changeBtnTitle(title){
            this.btnTitle = title;
            console.log('btn title changed');
        }

    },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>