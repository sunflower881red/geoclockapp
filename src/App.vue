<template>
  <div class="app" id="app">
    <div id="main"></div>
    <div @click="kakusi" class="kakusi"></div>
    <div id="back">
    </div>
        <div class="time_warpper animate__animated animate__slideInRight">
      <div class="day_ttl">{{ time.Year }} {{ time.Month }} {{ time.Day }}</div>
      <div class="time_ttl">{{ kazu }}</div>
    </div>
    <div v-if="!dayon" class="animate__animated animate__slideInRight txt_wrapper">
      <p v-if="tenkion && tenki.weather[0].main === 'Rain'" class="W_icon"><i class="bi bi-cloud-rain-fill"></i></p>
      <p v-if="tenkion && tenki.weather[0].main === 'Snow'" class="W_icon"><i class="bi bi-cloud-snow-fill"></i></p>
      <p v-if="tenkion && tenki.weather[0].main === 'Clear'" class="W_icon"><i class="bi bi-brightness-high-fill"></i></p>
      <p v-if="tenkion && tenki.weather[0].main === 'Clouds'" class="W_icon"><i class="bi bi-cloudy-fill"></i></p>
      <p v-if="tenkion" class="txt_ttl2">{{ tenki.weather[0].description }}</p>
      <p v-if="tenkion" class="txt_ttl">{{ tenki.name }}</p>
      <p v-if="tenkion" class="txt_ttl2">気温:{{ tenki.main.temp }}C°</p>
      <p v-if="tenkion" class="txt_ttl2">最高気温:{{ tenki.main.temp_max }}C°</p>
      <p v-if="tenkion" class="txt_ttl2">最低気温:{{ tenki.main.temp_min }}C°</p>
      <p v-if="tenkion" class="txt_ttl2">湿度:{{ tenki.main.humidity }}%</p>
    </div>
    <div v-if="dayon" class="animate__animated animate__slideInLeft txt_wrapper">
      <p class="txt_ttl">Weather from today to tomorrow</p>
      <div class="wea_list">
      <li v-for="yoho in yohou" :key="yoho.index">
        <div class="wea_contents">
          <p v-if="tenkion && yoho.weather[0].main === 'Rain'" class="W_icon2"><i class="bi bi-cloud-rain-fill"></i></p>
          <p v-if="tenkion && yoho.weather[0].main === 'Snow'" class="W_icon2"><i class="bi bi-cloud-snow-fill"></i></p>
          <p v-if="tenkion && yoho.weather[0].main === 'Clear'" class="W_icon2"><i class="bi bi-brightness-high-fill"></i></p>
          <p v-if="tenkion && yoho.weather[0].main === 'Clouds'" class="W_icon2"><i class="bi bi-cloudy-fill"></i></p>
          <p class="txt_ttl2">{{yoho.dt_txt.split(' ')[1].substr(0,5)}}</p>
          <p class="txt_ttl2">{{ yoho.weather[0].description }}</p>
        </div>
      </li>
    </div>
    </div>
    <div class="ttl_wrapper">
    <button  v-show="!this.dayon" @click="daychange()" class="arr_btn">Realtime Mode<i class="bi bi-arrow-repeat"></i></button>
    <button  v-show="this.dayon" @click="daychange()" class="arr_btn">Forecast Mode<i class="bi bi-arrow-repeat"></i></button>
    <button @click="getWeather()" class="ttl">Update Weather<i class="bi bi-cloud-download"></i></button>
    <button v-if="hidden" @click="Ramdom()" class="ttl">Ramdom</button>
     </div>
  </div>
</template>

<script>
import axios from 'axios';
import 'animate.css';
  export default {
    data: () => ({
     rr:'',
     kazu:'',
     time:{
      Year:'',
      Month:'',
      Day:''
     },
     geoOb:{},
     tenki:{},
     yohou:{},
     hidden:false,
     dayon:false,
     testdata:{
      lati:36.380149,
      long:126.732293
     },
     tenkion:false
    }),
    mounted(){
      this.Date();
      setInterval(this.Date,1000)
      setInterval(this.getWeather,180000)
      this.geo();   
    },
    created(){
      
    },
    methods:{
      Date(){
        let Time = new Date();
        let TimeH = Time.getHours();
        TimeH=this.Two(TimeH);
        let TimeM = Time.getMinutes();
        TimeM=this.Two(TimeM);
        let TimeS = Time.getSeconds();
        TimeS=this.Two(TimeS);
        this.kazu=`${TimeH}:${TimeM}:${TimeS}`;
        this.time.Year=Time.getFullYear();
        this.time.Month=Time.getMonth();
        this.time.Day=Time.getDate();
      },
      Two(i){
        let v = i;
        if( i < 10 )v = '0' + i;
        return v;
      },
      geo(){
              navigator.geolocation.getCurrentPosition((position)=> {
              this.geoOb=position.coords;
              this.getWeather();
              
          })
      },
      Ramdom(){
        let randlati = Math.random()*90-90;
        let randlong = Math.random()*180-180;
        console.log(randlati+" "+randlong);
        this.geoOb.latitude=randlati;
        this.geoOb.longitude=randlong;
        this.getWeather();
      }
      ,
      daychange(){
        this.dayon=!this.dayon;
      },
      kakusi(){
        this.hidden=!this.hidden;
      },
      backG(){
        const id = document.getElementById("main");
        const bid = document.getElementById("back");
        id.classList='';
        bid.classList='';
        let Time = new Date();
        let TimeH = Time.getHours();
        
          if(this.tenki.clouds.all<=10)id.classList.add('bluesky');
          if(this.tenki.clouds.all>10&&this.tenki.clouds.all<=40){
            id.classList.add('sun');
          }else if(this.tenki.clouds.all>40&&this.tenki.clouds.all<=80){
            id.classList.add('cloud');
          }else if(this.tenki.clouds.all>80&&this.tenki.clouds.all<=100){
            id.classList.add('cloud2');
          }
        if(TimeH>18){
            id.classList.add('night');
        }

         if(this.tenki.weather[0].main=='Rain'){
          bid.classList.add('box');
          id.classList.add('Rainy');
         }
         if(this.tenki.weather[0].main=='Snow'){
          bid.classList.add('snowbox');
          id.classList.add('Rainy');
         }
        
      },
      getWeather() {
        if(this.geoOb.latitude==null)this.geoOb.latitude=this.testdata.lati;
        if(this.geoOb.longitude==null)this.geoOb.longitude=this.testdata.long;
      axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${this.geoOb.latitude}&lon=${this.geoOb.longitude}&appid=8f525c97a23deffcfb6a9b6602d93015&lang=ja&units=metric`)
      .then(response => {
        this.tenki=response.data;
        this.tenkion=true;
        console.log(response.data);
        if(this.tenki.name=="")this.tenki.name="不明";
        this.backG();
      })
      axios.get(`https://api.openweathermap.org/data/2.5/forecast?lat=${this.geoOb.latitude}&lon=${this.geoOb.longitude}&appid=8f525c97a23deffcfb6a9b6602d93015&lang=ja&units=metric&cnt=8`)
      .then(response => {
        this.yohou=response.data.list.slice(response.data.list.length-6,response.data.list.length);
        console.log(response.data);
      })
      }
    },
    computed:{
    }
  }
</script>

<style>
@import "the-new-css-reset/css/reset.css";
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@500&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Quantico:wght@700&display=swap');
@import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.8.0/font/bootstrap-icons.css");
li{
  list-style: none;
}
#main{
  margin: 0;
  padding: 0;
  border: 0;
  position: absolute;
  width: 100vw;
  height: 100vh;
  z-index: -20;
  pointer-events: none;
  filter: brightness(0.8);
}
.kakusi{
  margin: 0;
  padding: 0;
  border: 0;
  position: absolute;
  width: 20px;
  height: 20px;
  z-index: 6;
}
#main .Rainy{
  filter: grayscale(30%);
}
#app{
  margin: 0;
  padding: 0;
  border: 0;
  position: relative;
  z-index: 0;
  background-color:#000;
  overflow: hidden;
}

.bluesky{
  background-image: url(backG/dave-hoefler-ELXbHhzVFO0-unsplash.webp);
  background-size: 140vw 100vh;
}
.sun{
  background-image: url(backG/x-xtgONQzGgOE-unsplash.webp);
  background-size: 140vw 100vh;
}
.cloud{
  background-image: url(backG/patrick-fore-V6s1cmE39XM-unsplash.webp);
  background-size: 140vw 100vh;
}
.cloud2{
  background-image: url(backG/tom-barrett--bSucp2nUdQ-unsplash.webp);
  background-size: 140vw 100vh;
}
.night{
  background-image: url(backG/ganapathy-kumar-ve_uN9V8xqU-unsplash.webp);
  background-size:500vw 200vh;
}

.time_warpper{
  margin: 0;
  padding: 0;
  border: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  width: 100vw;
  height: 40vh;
}
.time_ttl{
  font-size:min(11.5vw,170px);
  margin-top: 10px;
  color: white;
  font-family: 'Roboto', sans-serif;
}
.W_icon{
  font-size:min(14.5vw,130px);
  color: white;
  padding-bottom: 10px;
  font-family: 'Roboto', sans-serif;
}
.W_icon2{
  font-size:min(6.5vw,40px);
  color: white;
  padding-bottom: 10px;
  font-family: 'Roboto', sans-serif;
}
.day_ttl{
  font-size:min(6.5vw,60px);
  color: white;
  font-family: 'Roboto', sans-serif;
}
.ttl_wrapper{
  margin: 0;
  padding: 0;
  border: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  width: 100vw;
  height: 20vh;
}
.ttl{
  height: 10vh;
  margin-left:10px;
  margin-right:10px;
  padding-right: 10px;
  padding-left: 10px;
  border: solid 1px;
  border-radius: 10px;
  font-size: 3.5vh;
  color: rgba(240, 248, 255, 0.344);
  font-family: 'Quantico', sans-serif;
  transition: all 0.2s ease-out;
  justify-content: center;
  display: flex;
  align-items: center;
  cursor:pointer;
}
.arr_btn{
  font-size: 3rem;
  padding-right: 10px;
  padding-left: 10px;
  height: 10vh;
  margin-left:10px;
  margin-right:10px;
  border: solid 1px;
  border-radius: 10px;
  font-size: 3.5vh;
  color: rgba(240, 248, 255, 0.344);
  fill: rgba(240, 248, 255, 0.344);
  font-family: 'Quantico', sans-serif;
  transition: all 0.2s ease-out;
  justify-content: center;
  display: flex;
  align-items: center;
  cursor:pointer;
}
.arr_btn:hover{
  color: aliceblue;
}
.arr_btn:active{
  color: rgb(42, 156, 255);
}
.txt_wrapper{
  margin: 0;
  padding: 0;
  border: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  width: 100vw;
  height: 40vh;
}
.txt_ttl{
  color: aliceblue;
  font-size: 3.5vh;
  font-family: 'Quantico', sans-serif;
  
}
.txt_ttl2{
  color: aliceblue;
  font-size: 2.5vh;
  font-family: 'Quantico', sans-serif;
  
}
.ttl:hover{
  color: aliceblue;
}
.ttl:active{
  color: rgb(42, 156, 255);
}
.snowbox {
  position   : absolute;
  width  : 100vw;                  
  height     : 100vh;
  z-index: 2222;
  mix-blend-mode: lighten;
  filter: contrast(200%);
  pointer-events: none;
}
.snowbox::after {
  display    : block;
  content    : "";
  position   : absolute;
  top        : 0;
  right      : 0;
  bottom     : 0;
  left       : 0;
  background-image : url(backG/snow.png); 
  background-repeat: repeat;
  background-size: 200px 800px; 
  animation  : bgAnime 35s linear infinite;
  opacity: 30%;
}
.snowbox::before {
  display    : block;
  content    : "";
  position   : absolute;
  top        : 0;
  right      : 0;
  bottom     : 0;
  left       : 0;
  background-image : url(backG/snow.png); 
  background-repeat: repeat;
  background-size: 200px 800px; 
  animation  : bgAnime 45s linear infinite;
  opacity: 30%;
}
.box {
  position   : absolute;
  width  : 100vw;                  
  height     : 100vh;
  z-index: 2222;
  filter: contrast(200%);
  mix-blend-mode: lighten;
  pointer-events: none;
}
.box::after {
  display    : block;
  content    : "";
  position   : absolute;
  top        : 0;
  right      : 0;
  bottom     : 0;
  left       : 0;
  background-image : url(backG/rain.webp); 
  background-repeat: repeat;
  background-size: 200px 800px; 
  animation  : bgAnime2 25s linear infinite;
  opacity: 10%;
}
.box::before {
  display    : block;
  content    : "";
  position   : absolute;
  top        : 0;
  right      : 0;
  bottom     : 0;
  left       : 0;
  background-image : url(backG/rain.webp); 
  background-repeat: repeat;
  background-size: 200px 800px; 
  animation  : bgAnime 25s linear infinite;
  opacity: 20%;
}
.wea_list{
  width: 100vw;
  height: 100%;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}
.wea_contents{
  width: 100px;
  display: flex;
  justify-content: center;
  flex-direction: column;
}
@keyframes bgAnime {
   0% { background-position: 0 0 }
   100% { background-position: 400px 12800px }
}
@keyframes bgAnime2 {
  0% { background-position: 0 0 }
  100% { background-position: -400px 12800px }
}
</style>
