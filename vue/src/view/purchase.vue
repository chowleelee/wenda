<template>
  <div class="purchase" >
    <mt-navbar v-model="selected" :fixed="true">
      <div class="container">
        <mt-tab-item id="purchase-ask">我问</mt-tab-item>
        <mt-tab-item id="purchase-quickask">快问</mt-tab-item>
        <mt-tab-item id="purchase-listen">我看</mt-tab-item>
      </div>
    </mt-navbar>
    <!-- tab-container -->
    <mt-tab-container v-model="selected">
      <!-- 我问 -->
      <mt-tab-container-item id="purchase-ask" class="purchase-ask">
        <div class="myask">
          <span class="ask-num">共{{ask_list.length}}个问题</span>
          <div class="drop-wrap" @click = "ask_actived = true">
            <span class="drop-btn">{{ask_status[ask_current_status]}}</span>
            <i class="icon-arrow" :class="{'active':ask_actived}"></i>
            <div class="drop-menu" v-if="ask_actived">
              <div class="arrow-up"></div>
              <div v-for="(item,key) in ask_status" class="drop-item" :class="{current:key == ask_current_status}" @click.stop="changeAskStatus(key)">{{item}}</div>
            </div>
          </div>
        </div>
        <div class="tips" v-if="all_ask.length == 0">
          <p>你还没有提过问哟～</p>
          <router-link to='/findExpert' class="tip-btn">
            去找人页看看
          </router-link>
        </div>
        <mt-cell class="ask-list"  v-if="ask_list.length != 0">
          <ul>
            <li class="problemDetails-question" v-for='(item,index) in ask_list'>
              <div class="question-content"  >
                问题：{{item.ask_content}}
              </div>
              <div class="question-answer">
                <router-link :to="'/tutorDetails/'+item.vip_id" v-if="item.status==2">
                  <div class="problemDetails-avatar">
                    <img :src="$sourceUrl+'/img/'+item.head_img">
                    <img src="../assets/img/listen-answer-avatar-right.jpg" class="avatar-true">
                  </div>
                </router-link>
                 <div class="has-answer" v-if="item.status==2">
                    <my-text :content="item.answer_content"></my-text>
                 </div>
                 <div class="no-answer" v-if="item.status!=2">
                  暂无回答
                </div>
                <div class="status-tip" v-if="ask_current_status == 0">
                  {{ask_status[item.status]}}
                </div>
              </div>
              <div class="question-footer">
                <span class="time"><timeago :since="parseInt(item.create_time)*1000"></timeago></span>
                <div class="footer-count" v-if="item.listen_ask">
                  <span class="count">
                    <span>浏览</span>
                    <span>{{item.listen_num}}</span>
                  </span>
                  <span class="count" @click="toLike(index)">
                    <i class="like-icon" v-if="!item.like"></i>
                    <i class="like-icon-zan" v-else></i>
                    <span>{{item.like_num}}</span>
                  </span>
                </div>
              </div>
            </li>
          </ul>
        </mt-cell>
      </mt-tab-container-item>
      <!-- 快问 -->
      <mt-tab-container-item id="purchase-quickask" class="purchase-quickask">
        <div class="myask">
          <span class="ask-num">共{{qa_list.length}}个问题</span>
          <div class="drop-wrap" @click="quickly_actived = !quickly_actived">
            <span class="drop-btn">{{quickly_status[quickly_current_status]}}</span>
            <i class="icon-arrow" :class="{'active':quickly_actived}"></i>
            <div class="drop-menu" v-if="quickly_actived">
              <div class="arrow-up"></div>
              <div v-for="(item,key) in quickly_status" class="drop-item" :class="{current:key == quickly_current_status}" @click.stop="changeQuicklyStatus(key)">{{item}}</div>
            </div>
          </div>
        </div>
        <div class="tips" v-if="all_quickly.length == 0">
          <p>你还没有发布过快问哟～</p>
          <router-link to='/ask-quickly' class="tip-btn">
            去快问页看看
          </router-link>
        </div>
        <mt-cell  class="qa-list" v-if="qa_list.length != 0">
          <div class="rewards">
            <ul class="rewards-list">
              <li v-for="(item,index) in qa_list">
                <div class="rewards-content">
                  <p @click="my_quickask(item.id)">{{item.content}}</p>
                </div>
                <div class="rewards-footer">
                  <span v-if="item.status==1">还剩{{[item.create_time,item.duration]|remainTime}}小时</span>
                  <span v-if="item.status==2">已解决</span>
                  <span v-if="item.status==3">已过期</span>
                  <span v-if="item.status==4">已关闭</span>
                  <span v-if="item.status==1">{{ask_quickly_answer_num[quickly_status_index[index]]}}人已抢答</span>
                  <span v-if="item.status!=1">{{ask_quickly_answer_num[quickly_status_index[index]]}}人被选中</span>
                  <span class="status-tip" v-if="quickly_current_status == 0">
                    {{quickly_status[item.status]}}
                  </span>
                </div>
                <div class="rewards-tip" v-if="ask_quickly_answer_num[index] !=0">

                  <div class="info" v-for="(answer_item,k) in ask_quickly_answer">

                   <span class="tutor-info" v-if="k == quickly_status_index[index]">{{answer_item.real_name}}｜ {{answer_item.identity}} </span>
                   <span class="qiang" v-if="k == quickly_status_index[index]">{{ask_quickly_answer_time[quickly_status_index[index]]}}抢答</span>   
                 </div>
               </div>
             </li>
           </ul>
         </div>
        </mt-cell>        
      </mt-tab-container-item>
      <!-- 我看 -->
      <mt-tab-container-item id="purchase-listen" class="purchase-listen">
        <div class="tips" v-if="sb_list.length == 0">
          <p>你还没有偷看过问题哟～</p>
          <router-link to='/findExpert' class="tip-btn">
            去找人页看看
          </router-link>
        </div>
        <mt-cell class="sb-list" v-if="sb_list.length != 0">
          <ul class="tutorDetails-questions" >
            <li> 
              <div class="myask">
                <span class="ask-num">共{{sb_list.length}}个问题</span>
              </div>
            </li>
            <li class="a-question" v-for="(item,index) in sb_list">
              <div class="Headlines">
                <img :src="$sourceUrl+'/img/'+item.user_head" class="avatar">
                <router-link :to="'/problemDetails/'+item.id">
                  <p class="Headlines-content content">{{item.ask_content}}</p>
                  <p class="offer">￥{{item.price}}</p>
                </router-link>
              </div>
              <div class="question-answer">
                <router-link :to="'/tutorDetails/'+item.vip_id">
                  <img :src="$sourceUrl+'/img/'+item.head_img" class="avatar">
                </router-link>
                <my-text :content="item.answer_content"></my-text>
              </div>
              <div class="question-footer">
                <span class="time"><timeago :since="parseInt(item.create_time)*1000"></timeago></span>
                <div class="footer-count">
                  <span class="count">
                    <span>浏览</span>
                    <span>{{item.listen_num}}</span>
                  </span>
                  <span class="count" @click="toLikeListen(index)">
                    <i class="like-icon" v-if="!sb_like[index]"></i>
                    <i class="like-icon-zan" v-else></i>
                    <span>{{item.like_num}}</span>
                  </span>
                </div>
              </div>
            </li>
          </ul>
        </mt-cell>
      </mt-tab-container-item>
    </mt-tab-container>
  </div>
</template>

<script type="es6">

import {mapState,mapMutations,mapGetters} from 'vuex'
import {Toast} from 'mint-ui'
import myText from '@/components/text';
import { Indicator } from 'mint-ui'
  export default {
    data () {
      return {
        selected:"purchase-ask",
        ask_actived:false,
        quickly_actived:false,
        ask_current_status:0,
        quickly_current_status:0,
        ask_list:[],
        all_ask:[],
        no_answer_ask:[],
        answer_ask:[],
        timeOut_ask:[],
        refuse_ask:[],
        ask_status:['全部','待回答','已回答','已过期','被婉拒'],
        qa_list:[],
        all_quickly:[],
        underway_quickly:[],
        answered_quickly:[],
        timeOut_quickly:[],
        close_quickly:[],
        quickly_status:['全部','正在进行','已解答','已过期','已关闭'],
        quickly_status_index:[],
        ask_quickly_answer:[],
        ask_quickly_answer_num:[],
        ask_quickly_answer_time:[],
        ask_quickly_num:0,
        sb_list:[],
        like:[],
        sb_like:[],
      }
    },
    created(){
      this.nTime=parseInt(new Date().getTime()/1000);
      this.get_ask()
    },
    computed:{
        ...mapState(['info']), //把vuex要用到的状态映射出来 
        ...mapGetters(['isLogin'])
    },
    watch:{
      // ask_isplay:function(newVal){
      //   if(this.ask_list[newVal].flag == false){
      //     this.$http
      //     .post('askListen',{uid:this.info.user_id,askId:this.ask_list[newVal].id,vip_id:this.ask_list[newVal].vip_id})
      //     .then(()=>{
      //       this.ask_list[newVal].listen_num = ++this.ask_list[newVal].listen_num;
      //       this.ask_list[newVal].flag = true;
      //       this.$set(this.ask_list,newVal,this.ask_list[newVal])
      //     });
      //   }
      // },
      selected:function(newVal){
        if(newVal == 'purchase-ask' && this.ask_list.length==0){
          this.get_ask()
        }else if(newVal == 'purchase-quickask' && this.all_quickly.length==0){
          this.get_qa()
        }else if(newVal == 'purchase-listen' && this.sb_list.length==0){
          this.get_sb()
        }
      }
    },
    filters: {
      remainTime:function([value,duration]){
        var nowTime = new Date().getTime();
        var time = parseInt((nowTime - value*1000)/1000/60/60);
        var remainTime = duration - time;
        return remainTime = remainTime>0?remainTime:0;
      }
    },
    methods:{
      get_ask:function() {
        if(this.isLogin){
          Indicator.open();
          this.$http
          .get('purchaseask',{params:{id:this.info.user_id}})
          .then(rtnData=>{
            this.ask_list = rtnData.data 
            for(var i=0;i<this.ask_list.length;i++){
                if(this.ask_list[i].status == 1){
                  this.ask_list[i].listen_ask = false;
                  this.$set(this.ask_list,i,this.ask_list[i])
                  this.no_answer_ask.push(this.ask_list[i]);
                }else if(this.ask_list[i].status == 2){
                  this.ask_list[i].listen_ask = true;
                  this.$set(this.ask_list,i,this.ask_list[i])
                  this.answer_ask.push(this.ask_list[i]);
                }else if(this.ask_list[i].status == 3){
                  this.timeOut_ask.push(this.ask_list[i]);
                }else if(this.ask_list[i].status == 4){
                  this.refuse_ask.push(this.ask_list[i]);
                }
                let index = i;
                this.$http
                .get('/askLike/'+this.ask_list[i].id,{params:{uid:this.info.user_id}})
                .then(rtnData=>{
                  if(rtnData.data == null){
                    this.ask_list[index].like = false;
                    this.$set(this.ask_list,index,this.ask_list[index])
                  }else{
                    this.ask_list[index].like = true;
                    this.$set(this.ask_list,index,this.ask_list[index])
                  }
                });
                this.$http
                .get('/askListen/'+this.ask_list[i].id,{params:{uid:this.info.user_id}})
                .then(rtnData=>{
                  if(rtnData.data == null){
                    this.ask_list[index].flag = false;
                    this.$set(this.ask_list,index,this.ask_list[index])
                  }else{
                    this.ask_list[index].flag = true;
                    this.$set(this.ask_list,index,this.ask_list[index])
                  }
                }); 
                this.all_ask.push(this.ask_list[i]);
            }
            Indicator.close();
         })
        }else{
          Toast("请先登录");
        }        
      },
      get_qa:function() {
        if(this.isLogin){
          Indicator.open();
          this.$http
          .get('purchasequickask',{params:{id:this.info.user_id}})
          .then(rtnData=>{
            if(!this.all_quickly.length){
              this.ask_quickly_num = 0;
            }else{
              this.ask_quickly_num = this.all_quickly[0].length
            }

            for(var i=0;i<rtnData.data.length;i++){
              if(!this.all_quickly.length){
                this.all_quickly[0]=[rtnData.data[i]];
                this.all_quickly[1]=[this.ask_quickly_num+i];
              }else{
                this.all_quickly[0].push(rtnData.data[i]);
                this.all_quickly[1].push(this.ask_quickly_num+i);
              }
              if(rtnData.data[i].status == 1){
                if(!this.underway_quickly.length){
                  this.underway_quickly[0]=[rtnData.data[i]];
                  this.underway_quickly[1]=[this.ask_quickly_num+i];
                 }else{
                  this.underway_quickly[0].push(rtnData.data[i]);
                  this.underway_quickly[1].push(this.ask_quickly_num+i);
                 }
              }else if(rtnData.data[i].status == 2){
                 if(!this.answered_quickly.length){
                  this.answered_quickly[0]=[rtnData.data[i]];
                  this.answered_quickly[1]=[this.ask_quickly_num+i];
                 }else{
                  this.answered_quickly[0].push(rtnData.data[i]);
                  this.answered_quickly[1].push(this.ask_quickly_num+i);
                 }
              }else if(rtnData.data[i].status == 3){
                if(!this.timeOut_quickly.length){
                  this.timeOut_quickly[0]=[rtnData.data[i]];
                  this.timeOut_quickly[1]=[this.ask_quickly_num+i];
                 }else{
                  this.timeOut_quickly[0].push(rtnData.data[i]);
                  this.timeOut_quickly[1].push(this.ask_quickly_num+i);
                 }
              }else if(rtnData.data[i].status == 4){
                if(!this.close_quickly.length){
                  this.close_quickly[0]=[rtnData.data[i]];
                  this.close_quickly[1]=[this.ask_quickly_num+i];
                 }else{
                  this.close_quickly[0].push(trtnData.data[i]);
                  this.close_quickly[1].push(this.ask_quickly_num+i);
                 }
              }
              this.ask_quickly_answer_num.push(0);
              this.ask_quickly_answer_time.push(0);
              let index = i;
               this.$http.get("/quickaskAnswer/"+rtnData.data[i].id).then(rtnData=>{
                if(rtnData.data.length!=0){
                  var time = parseInt((rtnData.data[0].create_time -  this.all_quickly[0][index].create_time)*1000);
                  
                  var answerTime = '';
                  if(time/1000 > 60){
                    if(time/1000/60>60){
                      if(time/1000/60/60>24){
                        answerTime = Math.ceil(time/1000/60/60/24)+"天内";
                      }else{
                        answerTime = Math.ceil(time/1000/60/60)+"小时内";
                      }                   
                    }else{
                      answerTime = Math.ceil(time/1000/60)+'分钟内';
                    }
                  }else{
                    answerTime = '1分钟内';
                  }
                  this.$set(this.ask_quickly_answer_time,index,answerTime);
                  this.$set(this.ask_quickly_answer,index,rtnData.data[0])
                  this.$set(this.ask_quickly_answer_num,index,rtnData.data.length)
                }
              })
            }
            this.qa_list = this.all_quickly[0]||[];
            this.quickly_status_index = this.all_quickly[1]; 

            Indicator.close();
          })
        }else{
          Toast("请先登录");
        }        
      },
      get_sb:function() {
        if(this.isLogin){
          Indicator.open();
          this.$http
          .get('purchasebuy',{params:{id:this.info.user_id}})
          .then(rtnData=>{
              this.sb_list = rtnData.data 
              for(var i=0;i<this.sb_list.length;i++){
                let index = i;
                this.$http
                 .get('/askLike/'+this.sb_list[i].id,{params:{uid:this.info.user_id}})
                 .then(rtnData=>{
                    if(rtnData.data == null){
                      this.$set(this.sb_like,index,false)
                    }else{
                      this.$set(this.sb_like,index,true)
                    }
                  });
              } 
              Indicator.close();
            })
        }else{
          Toast("请先登录");
        }        
      },
      //切换语音
      // stopPlay:function(index){
      //   this.isplay = index;
      // },
      //切换语音
      // askStopPlay:function(index){
      //   this.ask_isplay = index;
      // },
      //切换问的状态
      changeAskStatus:function(index){
        this.ask_current_status = index;
        if(index == 0){
          this.ask_list = this.all_ask;
 
        }else if(index == 1){
          this.ask_list = this.no_answer_ask;
        }else if(index == 2){
          this.ask_list = this.answer_ask;
        }else if(index == 3){
          this.ask_list = this.timeOut_ask;
        }else if(index == 4){
          this.ask_list = this.refuse_ask;
        }
        this.ask_actived = false;
      },
      changeQuicklyStatus:function(index){
        this.quickly_current_status = index;
        if(index == 0){
          this.qa_list = this.all_quickly[0] || [];
          this.quickly_status_index =  this.all_quickly[1] || [];
        }else if(index == 1){
          this.qa_list = this.underway_quickly[0] || [];
          this.quickly_status_index =  this.underway_quickly[1] || [];
        }else if(index == 2){
          this.qa_list = this.answered_quickly[0] || [];
          this.quickly_status_index =  this.answered_quickly[1] || [];
        }else if(index == 3){
          this.qa_list = this.timeOut_quickly[0] || [];
          this.quickly_status_index =  this.timeOut_quickly[1] || [];
        }else if(index == 4){
          this.qa_list = this.close_quickly[0] || [];
          this.quickly_status_index =  this.close_quickly[1] || [];
        }
        this.quickly_actived = false;
      },
      my_quickask:function(n){
        this.$router.push('/my-quickask/'+n)
      },
      toLike:function(index){
        if(this.ask_list[index].like){
          return ;
        }
        this.ask_list[index].like_num +=1;
        this.ask_list[index].like = true;
        this.$set(this.ask_list,index,this.ask_list[index])
        this.$http
          .post('askLike',{uid:this.info.user_id,askId:this.ask_list[index].id})
          .then(rtnData=>{});
        this.$http
          .put('/ask/'+this.ask_list[index].id)
          .then(rtnData=>{});
     
      },
      toLikeListen:function(index){
        if(this.sb_like[index]){
          return ;
        }
        this.sb_list[index].like_num +=1;
        this.$set(this.sb_like,index,true);
        this.$http
          .post('askLike',{uid:this.info.user_id,askId:this.sb_list[index].id})
          .then(rtnData=>{});
        this.$http
        .put('/ask/'+this.sb_list[index].id)
        .then(rtnData=>{});
      }
    },
    components:{
       myText
    }
  }
  </script>

  <!-- Add "scoped" attribute to limit CSS to this component only -->
  <style >


  @import '../assets/css/purchase.css';


  </style>