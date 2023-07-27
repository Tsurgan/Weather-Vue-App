<script setup lang="ts">


</script>

<template>
  <div id="app" :class="typeof weather.elevation !='undefined' && current_temp > 16 ? 'warm': ''">
    <main>
      <div class="search-box">
        <input type="text" class="search-bar" placeholder="Поиск..." v-model="query" v-on:keypress="fetchWeather">
      </div>
    
      <div class="weather-wrap" v-if="typeof weather.elevation != 'undefined'">
        <div class="location-box">
          <div class="location">{{weather_name.name}}, {{ weather_name.state }}, {{ weather_name.country }}</div>
          <div class="date"> {{dateBuilder()}}</div>
        </div>
        <div class="weather-box">
          <div class="temp">{{current_temp}}°C</div>
          <div class="weather">{{current_weather}}</div>
        </div>
      </div>
    </main>



  </div>

</template>
<script>
export default {
  name: 'app',
  data(){
    return{
      api_key: 'c5232746fedf1647d080d06c21be2225',
      url_base: 'https://api.open-meteo.com/v1/',
      url_loc_base: 'http://api.openweathermap.org/geo/1.0/',
      query: '',
      lat: '',
      lon: '',
      weather: {},
      weather_name:{},
      current_temp: 0,
      current_time: '',
      current_weather: ''
    }
  },
  methods: {
    fetchWeather (e) {
      if (e.key =="Enter") {

        //direct?q=London&limit=5&appid={API key}

        fetch(`${this.url_loc_base}direct?q=${this.query}&limit=1&APPID=${this.api_key}`)
          .then(res => {
            return res.json();
          }).then(
            
          this.setLocResults);
      }

    },

    setLocResults(results) {
      console.log(results);
      this.lat=results[0].lat;
      this.lon=results[0].lon;
      this.weather_name=results[0];
      console.log(this.lat, this.lon);
      //
      fetch(`${this.url_base}forecast?latitude=${this.lat}&longitude=${this.lon}&hourly=temperature_2m&timezone=GMT&daily=weathercode`)
        .then(res => {
          return res.json();
        }).then(this.setResults);


      //
      

    },
    
    setResults(results) {
      console.log(results);
      var tzoffset = (new Date()).getTimezoneOffset() * 60000; //offset in milliseconds
      var localISOTime = (new Date(Date.now() - tzoffset)).toISOString().slice(0, -1);

      var basicISOTime = new Date(Date.now()).toISOString().slice(0, -1);

      //console.log(localISOTime);
      console.log(basicISOTime);
      this.weather = results;
      const weather_pattern = {
        clear:[0],
        cloudy:[1,2,3],
        fog:[45,48],
        drizzle:[51,53,55,56,57],
        rain:[61,63,64,66,67,80,81,82],
        snow:[71,73,75,77,85,86],
        thunderstorm:[95,96,99]
      }
      //var temp_weather=Object.keys(weather_pattern).find(key => weather_pattern[key] === this.weather.daily.weathercode[0]);
      var temp_weather=Object.keys(weather_pattern).find(key => weather_pattern[key].includes(this.weather.daily.weathercode[0]) );
      console.log(this.weather.daily.weathercode[0]);
      console.log(temp_weather);
      if (typeof temp_weather != 'undefined'){
        this.current_weather=temp_weather;
      }
      
      var leng =this.weather.hourly.time.length;
      //console.log(leng);
      for (let i = 0; i < leng-1; i++) {
        //console.log(this.weather.hourly.time[i])
        //console.log(localISOTime)
        //console.log(this.weather.hourly.time[i+1])
        if ((this.weather.hourly.time[i]<basicISOTime)&&(this.weather.hourly.time[i+1]>basicISOTime)){
            this.current_time=basicISOTime;
            this.current_temp=this.weather.hourly.temperature_2m[i];
            //console.log("break");
            break;
        }
      }
    },
    dateBuilder () {
      let d = new Date();
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();

      return `${day} ${date} ${month} ${year}`;
    }
  }

}
</script>
<style>
*{
  margin: 0;
  padding:0;
  box-sizing: border-box;

}
body {
  font-family: 'montserrat',sans-serif;

}
#app {
  background-image: url('./assets/cold-bg.jpg');
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}
#app.warm {
  background-image: url('./assets/warm-bg.jpg');
}
main {
  min-height: 100vh;
  padding: 25px;

  background-image: linear-gradient(to bottom,rgba(0,0,0,0.25),rgba(0,0,0,0.75));
}
.search-box {
  width: 100%;
  margin-bottom: 30px;

}
.search-box .search-bar {
  display:block;
  width:100%;
  padding:15px;
  color: #313131;
  font-size: 20px;
  appearance: none;
  border:none;
  outline:none;
  background:none;

  box-shadow: 0px 0px 8px rgba(0,0,0,0.25);
  background-color:rgba(255,255,255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;

}
.search-box .search-bar:focus{
  box-shadow: 0px 0px 16px rgba(0,0,0,0.25);
  background-color: rgba(255,255,255,0.75);
  border-radius: 16px 0px 16px 0px;

}
.location-box .location {
  color:#FFF;
  font-size:32px;
  font-weight: 500;
  text-align:center;
  text-shadow: 1px 3px rgba(0,0,0,0.25);
}
.location-box .date {
  color:#FFF;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align:center;
}
.weather-box {
  text-align:center;
}
.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color:#FFF;
  font-size: 102px;
  font-weight: 900;

  text-shadow: 3px 6px rgba(0,0,0,0.25);
  background-color: rgba(255,255,255,0.25);
  border-radius: 16px;
  margin:30px 0px;

  box-shadow: 3px 6px rgba(0,0,0,0.25);
}

.weather-box .weather {
  color:#FFF;
  font-size: 48px;
  font-weight: 700;
  font-style:italic;
  text-shadow: 3px 6px rgba(0,0,0.25);
}
</style>
