<template>
  <div style="width:100%;height:100%">
    <div class="recommend" v-if="isLogin" v-cloak>
      <ul>
        <goods-showcase :item="item" v-for="(item,index) in goodsList" :key="item.goods_id+index" class="recommend-item"></goods-showcase>
      </ul>
    </div>
    <un-login v-else v-cloak></un-login>
  </div>
</template>

<script>
  import {getRecommendData} from 'dao/recommend'
  import GoodsShowcase from 'components/GoodsShowcase/GoodsShowcase'
  import BScroll from 'better-scroll'
  import UnLogin from '../Login/UnLogin'
  import {mapActions,mapGetters} from 'vuex'
  export default {
    name: "Recommend",
    data(){
      return {
        goodsList: [],
        page: 1,
        count: 10
      }
    },
    components: {
      GoodsShowcase,
      UnLogin
    },
    mounted (){
      getRecommendData(this.page,this.count).then((val) => {
        this.goodsList = val.data.message;
      }),
      this.syncUserInfo(JSON.parse(sessionStorage.getItem("userInfo")))
    },
    methods:{
      ...mapActions([
        'syncUserInfo'
      ]),
      _initBScroll() {
      this.scrollView= new BScroll('.recommend', {
        scrollY: true,
        probeType: 3
      });
      this.scrollView.on('touchEnd', (pos) => {
        if(pos.y > 50){
          getRecommendData(this.page,this.count).then((val) => {
            this.goodsList = val.data.message
          })
        }
        if(pos.y < this.scrollView.maxScrollY - 20){
          getRecommendData(this.page,this.count).then((val) => {
            this.goodsList = this.goodsList.concat(val.data.message)
          })
        }
      })
      this.scrollView.on('scrollEnd', () => {
        this.scrollView.refresh();
      });
    },
  },
    watch:{
      goodsList() {
        this.$nextTick(() => {
          // 让当前的页码+1
          this.page += 1;
          this._initBScroll();
        })
      }
    },
    computed:{
      ...mapGetters([
        'isLogin'
      ])
    }
}
</script>

<style lang="stylus" ref="stylesheet/stylus">
  .recommend
    background #F5F5F5
    width 100%
    height 100%
    overflow hidden
    .recommend-item
      display flex
      flex-direction row
      flex-wrap wrap
      background #F5F5F5
      padding-bottom 50px
  [v-cloak]
    display none !important
</style>