<template>
  <div class="index" :class="{'overflow':isSearch}">
    <mysearch v-on:child-say="listenToChild"></mysearch>
    <mt-swipe>
      <mt-swipe-item v-for="item in slider">
        <img :src="$sourceUrl+'/img/'+item.slider_img" @click="toSmallTalkDetails(item.smalltalk_id)" alt="">
      </mt-swipe-item>
    </mt-swipe>
    <div class="content">
      <div class="menu">
        <ul>
          <li><router-link to='/findExpert'><img src="../assets/img/index_expert.png" alt=""><span>找专家</span></router-link></li>
          <li><router-link to='/ask-quickly'><img src="../assets/img/index_ask.png" alt=""><span>快问</span></router-link></li>
        </ul>
      </div>
      <div class="topLine">
        <div class="container">
          <h4 class="headline">头条 | 免费</h4>
          <ul class="topLine-list">
            <li @click="toHeadlineDetails(item.id)" v-for="(item,key) in headline_list">

              <img class="topLine-img" :src="$sourceUrl+'/img/'+item.heading_img" alt="" v-if="key == 0">
              <img class="topLine-play" src="../assets/img/index_play.png" alt="" v-if="key != 0">
              <div class="topLine-basicInfo">
                <p class="topLine-title">{{item.title}}</p>
                <span class="topLine-author" v-if="key == 0">{{item.real_name}} | {{item.identity}}</span> 
              </div>
            </li>
          </ul>
           <router-link to="/headline/all"><p class="unread">今日未读<span>{{nowday_headline_noreadnum}}</span></p></router-link>
         
        </div>
      </div>
    </div>


  </div>
</template>

<script type="es6">
  import {mapState,mapMutations,mapGetters} from 'vuex';
  import mysearch from '@/components/search'
  import { Indicator } from 'mint-ui'
  export default {
    data () {
      return {
        headline_list:[],
        nowday_headline_noreadnum:0,
        isSearch:false,
        slider:[]
      }
    },
    created(){
      this.init()
    },
    watch:{
    },
    computed:{
      ...mapState(['info']),
      ...mapGetters(['isLogin']) 
    },
    methods:{
      init:function(){
        Indicator.open();
        this.$http.get('slider').then(rtnData=>{
          this.slider = rtnData.data;
        });
        this.$http
          .get('/headline',{params:{id:'all'}})
          .then(rtnData=>{
               //console.log(rtnData)
              var nowday_headline_list_id = [];
              let nowDay = new Date();
              for(let i=0;i<rtnData.data.data.length;i++){
                let d = new Date(parseInt(rtnData.data.data[i].create_time) * 1000)
                
                //console.log(nowDay.getTime());
                if(d.getFullYear() == nowDay.getFullYear() && d.getMonth() ==nowDay.getMonth() && d.getDate() == nowDay.getDate()){
                  nowday_headline_list_id.push(rtnData.data.data[i].id)
                }
                if(i<3){
                   this.$set(this.headline_list,i,rtnData.data.data[i])
                }
               
              }
              if(this.isLogin){
                 //console.log(nowday_headline_list_id);
                 var nowday_headline_num = nowday_headline_list_id.length;
                 this.$http
                 .get('/headlineRead',{params:{id:this.info.user_id,type:'today'}})
                 .then(rtnData=>{
                  //console.log(rtnData.data)
                  for(let i=0;i<rtnData.data.length;i++){
                    if(nowday_headline_list_id.indexOf(rtnData.data[i].id)!=-1){
                      nowday_headline_num -- ;
                    }
                  }
                  //console.log(nowday_headline_num);
                  this.nowday_headline_noreadnum = nowday_headline_num;
                });
              }
              Indicator.close();
             
          });  
      },
      toHeadlineDetails:function(id){
        if(this.isLogin){
          this.$http
             .get('/headlineRead/'+id,{params:{uid:this.info.user_id}})
             .then(rtnData=>{
              if(rtnData.data == null){
                this.$http
                .post('/headlineRead',{'uid':this.info.user_id,'id':id})
                .then(rtnData=>{
                })
              }
            })
        }
        this.$router.push('/headline-details/'+id)
      },
      listenToChild:function(isSearch){
        this.isSearch = isSearch;
      }
    },
    components:{
      mysearch
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style >

@import '../assets/css/index.css';

</style>