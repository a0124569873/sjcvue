<template>  
    <div class="m_center2">     
        <div class="m_center">
          <!-- 地区选择器-->
          <v-distpicker @selected="onSelected" :province="temp.address__province" :city="temp.address__city" :area="temp.address__dist" ></v-distpicker>
          <button class="clearall-btn" v-on:click="clear"> 清空</button>
        </div> 
        <baidu-map class="bm-view" :center="center" :zoom="zoom" @ready="handler" :scroll-wheel-zoom="true"  :mapStyle="bmapStyle" >
           <!-- 加载标注-->
          <bml-marker-clusterer :averageCenter="true">
                <bm-marker v-for="(marker,index) of markers" :key="index" animation="BMAP_ANIMATION_BOUNCE" :position="{lng: marker.lng, lat: marker.lat}" v-on:click="Opencontent(marker)">
                </bm-marker>
          </bml-marker-clusterer>     
           <!--地图类型控件-->
          <bm-map-type :map-types="['BMAP_NORMAL_MAP','BMAP_SATELLITE_MAP', 'BMAP_HYBRID_MAP']" :offset="{width:450,height:8}" anchor="BMAP_ANCHOR_TOP_LEFT"></bm-map-type>
          <!-- 信息窗体-->
          <bm-info-window :position="{lng: infowindow.lng, lat: infowindow.lat}" :show="infowindow.show" @close="infoWindowClose" @open="infoWindowOpen">党员值班人:{{infowindow.uname}} <br>组别:{{infowindow.grouptype}}<br>值班地点:{{infowindow.address}}<br>值班时间:{{infowindow.worktime}}<br>值班口号:{{infowindow.content}}  </bm-info-window>>
          <bm-boundary name="长沙县" :strokeWeight="2" strokeColor="blue" fillColor=""  ></bm-boundary>
        </baidu-map>
    </div> 
</template>

<script>
import VDistpicker from '../main/Distpicker' //地区选择器组件
import { mapGetters } from 'vuex';
import mapStyle from '../../static/json/mapstyle.json';//地区json
import {BmlMarkerClusterer} from 'vue-baidu-map'
import mapvillage from '../../static/json/mapvillage';
export default {

components: { VDistpicker,BmlMarkerClusterer },//注册组件
name: "mainmap",
  data () {//局内数据
    return {
      temp: {
        address__province: '',//县
        address__city: '',//镇/街道
        address__dist: '',//村/社区
      },
      center: {lng: 0, lat: 0},  //地区中心点
      zoom: 11,//缩放等级
      infowindow:{lng:'',lat:'',address:'',content:'',uname:'',grouptype:'',worktime:'', show:false},//标注弹窗数据 值班信息
      bmapStyle: {
        styleJson:mapStyle        //地图个性样式
      },
      markers:[],//标注json数组
    }
  },
   methods: {//方法函数
      handler ({BMap, map}) { //地图初始化函数
      this.center.lng = 113.1
      this.center.lat = 28.240956
      this.zoom = 11
    },
    onSelected(data) { //地区选择器函数
      this.temp.address__province = data.province.value,
      this.temp.address__city = data.city.value,
      this.temp.address__dist = data.area.value,

      this.$store.commit('SET_SELECTPLACE',this.temp.address__dist[0]);
      this.center.lng = this.temp.address__dist[1],
      this.center.lat = this.temp.address__dist[2],
      this.zoom = 17
    },
    
    clear:function(event){ //地区清除按钮
      this.temp.address__province=''
      this.temp.address__city = ''
      this.temp.address__dist = ''
      
    },
    get:function(){ //3000个终端点
            //发送get请求
            this.$http.get('http://www.teavamc.com/api/pwork/five').then(function(res){
              
                          for (let i = 0; i < res.data.data.length; i++) {    

                            for(let j=0;j<mapvillage.length;j++){
                              if(mapvillage[j].address==res.data.data[i].address) //与地区json匹配赋予经纬度
                              {
                                
                                const pworkdata={address:res.data.data[i].address,uname:res.data.data[i].uname,grouptype:res.data.data[i].grouptype,worktime:res.data.data[i].worktime,content:res.data.data[i].content,lng: mapvillage[j].lng, lat: mapvillage[j].lat}                            
                                this.markers.push(pworkdata)
                              }     
                            }
                          }
                          
                        },function(){
                    console.log('请求失败处理');
                });
    },
    infoWindowClose:function () { //关闭点信息窗口
      this.infowindow.show = false
    },
    infoWindowOpen:function () { //打开点信息窗口
      this.infowindow.show = true
    },
    Opencontent:function (message) {//打开点信息窗口传值
      this.infowindow={lng:message.lng,lat:message.lat,address:message.address,content:message.content,uname:message.uname,grouptype:message.grouptype,worktime:message.worktime,show:true}
    },
    
    
    
   },
  computed: { //计算属性 取存在状态库中的值
     ...mapGetters(["themeName"]),
     listenstage(){ //返回状态库中的值
       return this.themeName;
     },
     
  },
  watch:{
      listenstage:function(vag){//实时监听状态库中值的改变 
        this.bmapStyle={styleJson:vag}
      }
  },
   mounted:function(){//页面初始化函数
        this.get();
    }

}
</script>
<style lang="scss">
.bm-view {
  width: 100%;
  height: 92vh;

}
.m_center2{
  position: relative;
  overflow: hidden;
  z-index:0;
}

.clearall-btn{
    font-size: 12px;
    height: 24px;
    border: #1968ab solid 1px;
    border-radius: 15px;
    padding: 1px 15px;
    color: #ffffff;
    background: #1968ab;
    margin: 0;
    text-align: center;
    
}
.anchorBL{
display:none;
}

</style>
