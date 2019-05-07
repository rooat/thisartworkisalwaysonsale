<template>
  <div class="container-outer">
    <div class="container-inner">
        <artcontent-before ref="artcontent-before" :language="language" v-show="isshow_before" :artprice="artprice" :etzprice="etzprice"
        :currentpatron= "currentpatron" :timeHeld = "timeHeld" :depositable="depositable" 
        :foreclosureTime="foreclosureTime" :totalCollected="totalCollected" :tokenType="tokenType" :artName="artName"/>

        <artcontent-after ref="artcontent-after" :language="language" v-show="isshow_after" :artprice="artprice"
        :etzprice="etzprice" :currentpatron= "currentpatron" :timeHeld = "timeHeld" 
        :depositable="depositable" :foreclosureTime="foreclosureTime" :totalCollected="totalCollected" :tokenType="tokenType" :artName="artName"/>
    </div>
  </div>
</template>

<script>
  import ArtContent1 from './Sections/ArtContent1';
  import ArtContent2 from './Sections/ArtContent2';
  import chainApi from '../service/ChainApi'
  export default {
    name: 'Main',
    components: {
      'artcontent-before': ArtContent1,
      'artcontent-after': ArtContent2,
    },
    data() {
      return {
        isshow_before:true,
        isshow_after:false,
        language: 'en',
        artprice :"0",
        etzprice:"0",
        currentpatron:"",
        timeHeld:"0",
        depositable:"0",
        foreclosureTime:"0",
        totalCollected:"0",
        tokenType:"以太零",
        artName:""
      };
    },
    mounted() {
      chainApi.Initial();
      this.init();
      
    },
    methods:{
      init:function(){

        this.$trans.$off("inithome")
        this.$trans.$on("inithome",(data)=>{
          console.log("data:",data.islogin)
            if(data.islogin==true){
              this.isshow_before = false;
              this.isshow_after= true;
            }else{
              this.isshow_before = true;
              this.isshow_after= false;
            }
            this.firstPrice();
            this.autoPrice();
            this.getData();
        })
        this.$trans.$off("operate");
        this.$trans.$on("operate",(data)=>{
          console.log("data===",data)
          this.getData();
        })
      },
      firstPrice:async function(){
        let price = await this.$maseter.methods.price().call();
        let etzprice = await this.$price.methods.currentPrice().call();
        this.artprice = price/10**18+" etz";
        this.etzprice = "¥ "+Number(etzprice*6.5*price/10**22).toFixed(2);
      },
      autoPrice: function(){
        setInterval(this.firstPrice,2000);
      },
      getData:async function(){
        let currentpatron = await this.$token.methods.ownerOf(42).call();
        let timeLastCollected = await this.$maseter.methods.timeLastCollected().call();
        let depositable = await this.$maseter.methods.depositAbleToWithdraw().call();
        let foreclosureTime = await this.$maseter.methods.foreclosureTime().call();
        let totalCollected = await this.$maseter.methods.totalCollected().call();
        let artName = await this.$maseter.methods.artName().call();
        this.currentpatron = currentpatron;
        this.timeHeld = parseInt((new Date().getTime()-timeLastCollected*1000)/86400000)+" 日";

        this.depositable = String(depositable/10**18).substr(0,4)+" etz";
        this.foreclosureTime = chainApi.timeFormat(new Date(foreclosureTime*1000));
        this.totalCollected = Number(totalCollected/10**18).toFixed(2)+" etz";
        this.artName = artName;
      }
    }
  };
</script>

<style lang="scss" scoped>
  @import '../styles/util.scss';
  @import '../styles/app.scss';

</style>
