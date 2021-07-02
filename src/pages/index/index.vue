<template>
<div class="wrapper">
  <div class="header-wrapper">
    <div class="header-title">
      <span>{{area}}-{{city}}</span>
    </div>
    <div class="header-text">
      <span>{{weather}}|空气-{{airText}}</span>
      <span>{{airValue}}</span>
    </div>
    <div class="weather-advice">{{weatherAdvice}}</div>
  </div>
  <div class="body-wrapper">
    <div class="body">
      <div class="data-wrapper">
        <div class="data">
          <img class="data-logo" src="/static/images/wendu.png"/>
          <div class="data-text">
            <div class="data-title">温度</div>
            <div class="data-value">{{Temp}}C</div>
          </div>
        </div>
        <div class="data">
          <img class="data-logo" src="/static/images/shidu.png"/>
          <div class="data-text">
            <div class="data-title">湿度</div>
            <div class="data-value">{{Hum}}%</div>
          </div>
        </div>
      </div>
       <div class="data-wrapper">
        <div class="data">
          <img class="data-logo" src="/static/images/guangzhaodu.png"/>
          <div class="data-text">
            <div class="data-title">光照度</div>
            <div class="data-value">{{Light}}Lx</div>
          </div>
        </div>
        <div class="data">
          <img class="data-logo" src="/static/images/keranqiti.png"/>
          <div class="data-text">
            <div class="data-title">可燃气体</div>
            <div class="data-value">{{Ppm}}ppm</div>
          </div>
        </div>
      </div>
       <div class="data-wrapper">
        <div class="data">
          <img class="data-logo" src="/static/images/deng.png"/>
          <div class="data-text">
            <div class="data-title">灯</div>
            <div class="data-value">
              <switch @change="onLedChange" :checked="Led" color="#ff7500"/>
            </div>
          </div>
        </div>
        <div class="data">
          <img class="data-logo" src="/static/images/chuanhu.png"/>
          <div class="data-text">
            <div class="data-title">窗户</div>
            <div class="data-value">
              <switch @change="onChuanChange" :checked="Chuan" color="#ff7500"/>
            </div>
          </div>
        </div>
      </div>
       <div class="data-wrapper">
        <div class="data">
          <img class="data-logo" src="/static/images/fengshan.png"/>
          <div class="data-text">
            <div class="data-title">风扇</div>
            <div class="data-value">
              <switch @change="onFengChange" :checked="Feng" color="#ff7500"/>
            </div>
          </div>
        </div>
        <div class="data">
          <img class="data-logo" src="/static/images/beep.png"/>
          <div class="data-text">
            <div class="data-title">报警状态</div>
            <div class="data-value">
              <switch @change="onBeepChange" :checked="Beep" color="#ff7500"/>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import {connect}from 'mqtt/dist/mqtt.js';

const mqttUrl ='wxs://mqtt.yybshuai.top:8084/mqtt'//服务器连接域名

export default {
  data () {
    return {  
      client:{},
      Temp:0,
      Hum:0,
      Light:0,
      Ppm:0,
      Led:false,
      Chuan:false,
      Feng:false,
      Beep:false,
      area:'...',//城区
      city:'...',//城市
      airText:'...',//空气质量
      airValue:0,//空气指数
      weather:'...',//天气
      weatherAdvice:'...'//天气建议
    };
  },

  components: {

  },

  methods: {
    onLedChange(event){
      var that = this;
      console.log(event.mp.detail);
      let sw = event.mp.detail.value;
      that.Led = sw;
      that
      if(sw){
        that.client.publish('/yyb/sub','{"object":"LED","value":1}',function(err){
          if(!err){
            console.log("开灯");
          }
        })
      }
      else{
          that.client.publish('/yyb/sub','{"object":"LED","value":0}',function(err){
          if(!err){
            console.log("关灯");
          }
        })
      }
    },
    onChuanChange(event){
      var that = this;
      console.log(event.mp.detail);
      let sw = event.mp.detail.value;
      that.Chuan = sw;
      that
      if(sw){
        that.client.publish('/yyb/sub','{"object":"Chuan","value":1}',function(err){
          if(!err){
            console.log("打开窗户");
          }
        })
      }
      else{
          that.client.publish('/yyb/sub','{"object":"Chuan","value":0}',function(err){
          if(!err){
            console.log("关闭窗户");
          }
        })
      }
    },
    onFengChange(event){
      var that = this;
      console.log(event.mp.detail);
      let sw = event.mp.detail.value;
      that.Feng = sw;
      that
      if(sw){
        that.client.publish('/yyb/sub','{"object":"Feng","value":1}',function(err){
          if(!err){
            console.log("打开风扇");
          }
        })
      }
      else{
          that.client.publish('/yyb/sub','{"object":"Feng","value":0}',function(err){
          if(!err){
            console.log("关闭风扇");
          }
        })
      }
    },
      onBeepChange(event){
      var that = this;
      console.log(event.mp.detail);
      let sw = event.mp.detail.value;
      that.Beep = sw;
      if(sw){
        that.client.publish('/yyb/sub','{"object":"BEEP","value":1}',function(err){
          if(!err){
            console.log("打开蜂鸣器");
          }
        })
      }
      else{
          that.client.publish('/yyb/sub','{"object":BEEP","value":0}',function(err){
          if(!err){
            console.log("关闭蜂鸣器");
          }
        })
      }
    }
  },

  //created () {
    // let app = getApp()
  //}
    onShow(){
    var that = this;
    that.client = connect(mqttUrl);
    that.client.on('connect',function () {
        console.log("1.mqtt服务器已连接！");
        that.client.subscribe("/yyb/pub",function (err){
          if(!err){
            console.log("2.设备上行数据已订阅！");
          }
        })
    })
    that.client.on('message',function(topic,message){
      console.log(topic)
      console.log(message)//message是16进制的Buffer字节流
      let dataFromDev = {}
      dataFromDev = JSON.parse(message)
      console.log(dataFromDev)
      that.Temp = dataFromDev.Temp
      that.Hum = dataFromDev.Hum
      that.Light = dataFromDev.Light 
      that.Led = dataFromDev.Led
      that.Ppm = dataFromDev.Ppm
      that.Chuan = dataFromDev.Chuan
      that.Feng = dataFromDev.Feng
      that.Beep = dataFromDev.Beep
    });
wx.getLocation({
  type: 'wgs84',
  success (res) {
   const latitude = res.latitude
   const longitude = res.longitude
   const key = 'b20cb619bd544ed28ff6a54c0b1aabdf'
      wx.request({
      url: `https://free-api.heweather.net/s6/weather/now?location=${longitude},${latitude}&key=${key}`, //获取天气API地址
      success (res) {
        //console.log(res.data)
        const { basic , now} = res.data.HeWeather6[0];
        //console.log(basic)
       // console.log(now)
        that.area = basic.location
        that.city = basic.parent_city
        that.weather = now.cond_txt
        wx.request({
        url: `https://free-api.heweather.net/s6/air/now?location=${that.city}&key=${key}`, //获取空气数据的api接口地址
          success (res) {
              //console.log(res.data);
              const { air_now_city } = res.data.HeWeather6[0];
              console.log(air_now_city);
              const { aqi, qlty } = air_now_city;
              that.airText = qlty;
              that.airValue = aqi;
          }
        });
      }
    });
      wx.request({
       url: `https://free-api.heweather.net/s6/weather/lifestyle?location=${longitude},${latitude}&key=${key}`, //获取空气数据的api接口地址
        success (res) {
            console.log(res.data);
            console.log(res.data);
            const { lifestyle} = res.data.HeWeather6[0];
            console.log(lifestyle)
            that.weatherAdvice = lifestyle[2].txt
        }
      });
 }
});
  }
};
</script>

<style lang="scss" scoped>
.wrapper{
  padding: 15px;
  .header-wrapper{
    background-color: #ff7500;
    border-radius: 15px;
    color: #fff;
    box-shadow: #d6d6d6 0px 0px 5px;
    padding: 10px 30px;
    .header-title{
      display: flex;
      justify-content: space-between;
    }
    .header-text{
      font-size: 28px;
      font-weight: 350;
      display: flex;
      justify-content: space-between;
    }
    .weather-advice{
      margin-top: 20px;
      font-size: 13px;
    }
  }
  .data-wrapper{
    margin-top: 12px;
    display: flex;
    justify-content: space-between;
    .data{
      background-color: #fff;
      width: 150px;
      height: 75px;
      border-radius: 20px;
      display: flex;
      justify-content: space-around;
      padding: 0 8px;
      box-shadow: #d6d6d6 0px 0px 5px;
      .data-logo{
        height: 36px;
        width: 36px;
        margin-top: 12px;
      }
      .data-text{
        margin-top: 12px;
        color: #7f7f7f;
        .data-title{
          text-align: right;
        }
        .data-value{
          font-size: 28px;
        }
      }
    }
  }
}
</style>
