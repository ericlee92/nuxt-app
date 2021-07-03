<template>
  <!-- <Tutorial/> -->

  <div>
    <div style="text-align:center">
      <h1>Weather Forecast</h1>
    </div>
    <b-card>
    <b-container fluid>
      <b-row>
        <b-col>
          <b-input-group >
            <!-- <v-select class="form-control" placeholder="Select cities" v-model="selected_city" :options="cityList"></v-select> -->
            <vue-simple-suggest style="width:60%" v-model="selected_city" :list="cityList" :filter-by-query="true" placeholder="Type to search cities" @select="search"></vue-simple-suggest>
            <template #append>
              <b-button variant="primary" @click="search">Search</b-button>
              <b-button variant="outline-primary" @click="getSelfLocation">Get my location</b-button>
            </template>
          </b-input-group>

          <transition name="slide-fade">
            <b-card v-show="flag_current_weather" header-tag="header" footer-tag="footer" no-body >
              <template #header>
                <h6 class="mb-0" style="color:black">{{ current_weather.name }} </h6>
              </template>
              <b-card-body >
                <b-row> 
                  <b-col>
                     <b-card-text style="display:flex">Mostly {{ current_weather.weather_main }} ({{ current_weather.weather_description }}) <img style="width:65px; height:65px" :src="current_weather.icon"></b-card-text>

                     <div style="display:flex">
                      <h1>
                        {{ formatTempValue(current_weather.temp) }}°C
                       <!-- <span style="font-size: 15px; position: absolute; margin-top: 5px;">°C</span> -->
                      </h1>
                      <div style="margin-left:40px">
                        <label style="display:block; color:black">Max {{ formatTempValue(current_weather.temp_max) }}°C</label>
                        <label style="display:block; color:black">Min {{ formatTempValue(current_weather.temp_min) }}°C</label>
                      </div>
                    </div>
                    <p>
                      <label style="display:block">Wind {{current_weather.wind_speed}} m/s.</label>
                      <label style="display:block">Clouds {{current_weather.clouds}} %</label>
                      <label style="display:block">Humidity {{current_weather.humidity}} %</label>
                      <label style="display:block">Visibility {{current_weather.visibility}} m</label>
                      <label style="display:block">Atmospheric pressure {{current_weather.pressure}} hPa</label>
                    </p>
                  </b-col>
                  <b-col>
                    <b-list-group>
                      <b-list-group-item button :key="index" v-for="(item,index) in forecast" @click="updateList(index)" :active="item.active === 'true' ">
                        <div class="d-flex w-100 justify-content-between">
                          <h6 class="mb-1">{{formatTime(item.dt)}} ({{formatDay(item.dt)}}) </h6>
                          <img style="width:40px; height:40px" :src="item.icon">
                        </div>
                        {{ item.temp }}°C
                      </b-list-group-item>
                    </b-list-group>
                  </b-col>
                </b-row>     
              </b-card-body>             
              
              <!-- <b-button href="#" variant="primary">Go somewhere</b-button> -->

              <template #footer style="text-align:center">
                <b-button variant="primary" @click="addFavourite">Add to watchlist</b-button>
              </template>
            </b-card>
          </transition>
        </b-col>
        <b-col>
          <b-carousel
            id="carousel-1"
            v-model="slide"
            :interval="90000"
            :controls="flag_carousel"
            :indicators="flag_carousel"
            background="#ababab"
            img-width="1024"
            img-height="550"
            style="text-shadow: 1px 1px 2px #333;"
            @sliding-start="onSlideStart"
          >
            <!-- Text slides with image -->
            <b-carousel-slide v-if="flag_carousel"
              v-for="(item,index) in forecast"
              :key="index"
              :caption="item.temp+'°C'"
              :text="formatTempValue(item.temp_min)+'°C/'+formatTempValue(item.temp_max)+'°C'"
              style="width:100%; max-height:550px"
              :img-src="'/'+item.weather_main+'.jpg'"
            >
              <h5>{{item.weather_main}}</h5>
              <small>{{item.weather_description}}</small>
            </b-carousel-slide>

            <!-- Slide with blank fluid image to maintain slide aspect ratio -->
            <b-carousel-slide v-if="!flag_carousel" caption="No Cities Selected" img-blank img-alt="Blank image">
              <!-- <p>
                Please select cities from the list
              </p> -->
            </b-carousel-slide>
          </b-carousel>
        </b-col>
      </b-row>
    </b-container>
    </b-card>
    
    <b-container v-if="favourite.length > 0">
      <h4>Favourite Watchlist</h4>
      <b-tabs card active-nav-item-class="font-weight-bold text-uppercase" active-tab-class="font-weight-bold">
        <b-tab no-body :title="item.name" :key="index" v-for="(item,index) in favourite" header-tag="header" footer-tag="footer" no-body>
          
          <b-card-body >
            <b-row> 
              <b-col cols="3">
                 <b-card-text style="display:flex">Mostly {{ item.weather_main }} ({{ item.weather_description }}) <img style="width:65px; height:65px" :src="item.icon"></b-card-text>

                 <div style="display:flex">
                  <h1>
                    {{ formatTempValue(item.temp) }}°C
                   <!-- <span style="font-size: 15px; position: absolute; margin-top: 5px;">°C</span> -->
                  </h1>
                  <div style="margin-left:40px">
                    <label style="display:block; color:black">Max {{ formatTempValue(item.temp_max) }}°C</label>
                    <label style="display:block; color:black">Min {{ formatTempValue(item.temp_min) }}°C</label>
                  </div>
                </div>
                <p>
                  <label style="display:block">Wind {{item.wind_speed}} m/s.</label>
                  <label style="display:block">Clouds {{item.clouds}} %</label>
                  <label style="display:block">Humidity {{item.humidity}} %</label>
                  <label style="display:block">Visibility {{item.visibility}} m</label>
                  <label style="display:block">Atmospheric pressure {{item.pressure}} hPa</label>
                </p>
              </b-col>
              <b-col cols="3">
                <b-list-group>
                  <b-list-group-item :key="index2" v-for="(item2,index2) in item.forecast">
                    <div class="d-flex w-100 justify-content-between">
                      <h6 class="mb-1">{{formatTime(item2.dt)}} ({{formatDay(item2.dt)}}) </h6>
                      <img style="width:40px; height:40px" :src="item2.icon">
                    </div>
                    {{ item2.temp }}°C
                  </b-list-group-item>
                </b-list-group>
              </b-col>
            </b-row>     
            <b-button variant="outline-danger" @click="remove(index)">Remove</b-button>
          </b-card-body>             
          <!-- <template #footer style="text-align:center">
            <b-button variant="outline-danger" @click="remove(index)">Remove</b-button>
          </template> -->
        </b-tab>
      </b-tabs>
    </b-container>
  </div>

</template>

<script>
import axios from 'axios'
export default {
  components: {
  },
  data () {
    return {
      slide: 0,
      selected_city: '',
      favourite: [],

      flag_carousel: false,
      flag_current_weather: false,

      cityList: [
        'Air Itam',
        'Balik Pulau',
        'Batu Ferringhi',
        'Batu Maung',
        'Bayan Lepas',
        'Bukit Mertajam',
        'Butterworth',
        'Gelugor',
        'Jelutong',
        'Kepala Batas',
        'Kubang Semang',
        'Nibong Tebal',
        'Penaga',
        'Penang Hill',
        'Perai',
        'Permatang Pauh',
        'Pulau Pinang',
        'Simpang Ampat',
        'Sungai Jawi',
        'Tanjung Bungah',
        'Tasek Gelugor',
        'USM Pulau Pinang',

        'Ampang',
        'Batu Caves',
        'Cheras',
        'Damansara',
        'Gombak',
        'Hulu Kelang',
        'Kepong',
        'Kuala Lumpur',
        'Petaling',
        'Petaling Jaya',
        'Sentul',
        'Setapak',
        'Sungai Besi',

        'Bahau',
        'Bandar Enstek',
        'Bandar Seri Jempol',
        'Batu Kikir',
        'Gemas',
        'Gemencheh',
        'Johol',
        'Kota',
        'Kuala Klawang',
        'Kuala Pilah',
        'Labu',
        'Linggi',
        'Mantin',
        'Nilai',
        'Port Dickson',
        'Pusat Bandar Palong',
        'Rantau',
        'Rembau',
        'Rompin',
        'Seremban',
        'Si Rusa',
        'Simpang Durian',
        'Simpang Pertang',
        'Tampin',
        'Tanjong Ipoh'
        ],

      carousel_view: {
        dt: '',
        clouds: '',
        wind_speed: '',
        temp: '',
        temp_min: '',
        temp_max: '',
        pressure: '',
        humidity: '',
        lon: '',
        lat: '',
        weather_id: '',
        weather_main: '',
        weather_description: '',
        weather_icon: '',
        icon: ''
      },
      forecast: [

      ],

      current_weather:{
        name: '',
        dt: '',
        clouds: '',
        wind_speed: '',
        visibility: '',
        temp: '',
        feels_like: '',
        temp_min: '',
        temp_max: '',
        pressure: '',
        humidity: '',
        lon: '',
        lat: '',
        weather_id: '',
        weather_main: '',
        weather_description: '',
        weather_icon: '',
        icon: ''
      }
    }
  },
  watch: {
    cityList: {
        handler: function () {
            console.log('caught!');
            localStorage.setItem("city_list", JSON.stringify(this.cityList))
        },
        deep: true
    }
    /*selected_city: function(){
      const self = this
      self.flag_carousel = false
      self.flag_current_weather = false
      if(self.selected_city !== '' ){
       self.verifyLatLng('','',self.selected_city)
      }
    }*/
  },
  computed: {
  },
  methods: {
    addFavourite(){
      const self = this
      if(self.favourite.length < 5){
        let flag_exist = false
        self.favourite.forEach(function(item){
          if(item.lat === self.current_weather.lat && item.lon === self.current_weather.lon){
            flag_exist = true
          }
        })

        if(flag_exist){
          alert('This city already added')
        }
        else{
          let obj = JSON.parse(JSON.stringify(self.current_weather))
          obj.forecast = self.forecast
          self.favourite.push(obj)
          localStorage.setItem("favourite", JSON.stringify(this.favourite))
        }

        
      }
      else{
        alert('reach 5 limit')
      }
    },
    remove(index){
      var permission = confirm('Are you sure to remove this?')
      if(permission){
        this.favourite.splice(index, 1)
        localStorage.setItem("favourite", JSON.stringify(this.favourite))
      }
    },
    search(){
      const self = this
      self.flag_carousel = false
      self.flag_current_weather = false
      // self.selected_city = self.selected_city.trim()
      setTimeout(function(){
        if(self.cityList.indexOf(self.selected_city) === -1 ){
          let temp = self.selected_city.trim().split(' ')
          let new_word = ''
          temp.forEach(function(item, index){
            if(index >= 1){
              new_word += ' '
            }
            //make first string uppercase
            new_word += item.charAt(0).toUpperCase() + item.slice(1)
          })

          self.cityList.push(new_word)
        }

        self.verifyLatLng('','',self.selected_city.trim())
      },300)

      
    },
    updateList(index){
      const self = this
      index = parseInt(index)
      self.forecast.forEach(function(item){
        item.active = 'false'
      })

      self.forecast[index].active = 'true'
      self.slide = index
      let chosen = index
      self.carousel_view = {
        dt: self.forecast[chosen].dt,
        clouds: self.forecast[chosen].clouds,
        wind_speed: self.forecast[chosen].wind_speed,
        temp: self.forecast[chosen].temp,
        temp_min: self.forecast[chosen].temp_min,
        temp_max: self.forecast[chosen].temp_max,
        pressure: self.forecast[chosen].pressure,
        humidity: self.forecast[chosen].humidity,
        lon: self.forecast[chosen].lon,
        lat: self.forecast[chosen].lat,
        weather_id: self.forecast[chosen].weather_id,
        weather_main: self.forecast[chosen].weather_main,
        weather_description: self.forecast[chosen].weather_description,
        weather_icon: self.forecast[chosen].weather_icon,
        icon: self.forecast[chosen].icon
      }
    },
    onSlideStart(slide){
      this.updateList(slide)
    },
    getNowTemp(obj){
      var today = new Date()
      var time = today.getHours()
      if(time >= 6 && time < 12){
        //day
        return this.formatTempValue(obj.day)
      }
      else if(time >= 12 && time < 18){
        //eve
        return this.formatTempValue(obj.eve)
      }
      else if(time >= 18 && time !== 0){
        //night
        return this.formatTempValue(obj.night)
      }
      else {
        //morn
        return this.formatTempValue(obj.morn)
      }
    },
    formatTempValue(data){
      if(parseInt(data)){
        return parseInt(data, 10)
      }
      else{
        return data
      }
      
    },
    formatTime(data){
      var date = new Date(data * 1000)
      return date.toISOString().split('T')[0]
    },
    formatDay(data){
      var date = new Date(data * 1000)
      var get_date = date.toISOString().split('T')[0]
      var temp = ['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday']

      return temp[new Date(get_date).getDay()]
    },
    
    //get self location long and lat
    getSelfLocation(){
      this.selected_city = ''
      navigator.geolocation.getCurrentPosition(this.success, this.error, {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0
      })
    },
    success(pos){
      console.log('success')
      var crd = pos.coords;
      this.flag_current_weather = false
      this.flag_carousel = false
      this.verifyLatLng(crd.latitude, crd.longitude)
    },
    error(){
      console.log('error')
    },

    //check parameter has lat and long, if only city then run google map api and return its lat & lng
    verifyLatLng(lat = '',lon = '', city = '') {
      const self = this
      console.log(lat,lon, city)
      if(lat === '' && lon === ''){
        let googlemap_api = `https://maps.googleapis.com/maps/api/geocode/json?address=${city}&key=AIzaSyBlWNrMEG5FRtNcNndrWHI9xw6FhKnbUk0`
        axios
          .get(googlemap_api)
          .then(response => {
            console.log('google map api => ', response)
            lat = response.data.results[0].geometry.location.lat
            lon = response.data.results[0].geometry.location.lng
            city = response.data.results[0].formatted_address
            self.getCurrentWeather(lat, lon, city)
        })
        .catch(error => {
          console.log(error);
        })
      }
      else{
        let googlemap_api = `https://maps.googleapis.com/maps/api/geocode/json?latlng=${lat},${lon}&key=AIzaSyBlWNrMEG5FRtNcNndrWHI9xw6FhKnbUk0`
        axios
          .get(googlemap_api)
          .then(response => {
            console.log('google map api => ', response)
            city = response.data.results[0].formatted_address
            self.getCurrentWeather(lat, lon, city)
            /*lat = response.data.results[0].geometry.location.lat
            lon = response.data.results[0].geometry.location.lng
            city = response.data.results[0].formatted_address
            self.getCurrentWeather(lat, lon, city)*/
        })
        .catch(error => {
          console.log(error);
        })

        self.getCurrentWeather(lat, lon, city)
      }          
    },
    getCurrentWeather(lat ,lon, city){
      const self = this
      let API_KEY = '41d92ff4dcbd190aa3cd2cfaa70797c7'
      let url = `http://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=metric&appid=${API_KEY}`
      let url2 = `https://api.openweathermap.org/data/2.5/onecall?lat=${lat}&lon=${lon}&exclude=minutely,current,hourly&units=metric&appid=${API_KEY}`

      if(city.length > 45){
        city = city.substring(0, 45)+"..."
      }
      //get current weather first
      axios
          .get(url)
          .then(response => {
            console.log('current result ',response)
            let current = response.data

            self.current_weather = {
              name: city,
              dt: current.dt,
              clouds: current.clouds.all,
              wind_speed: current.wind.speed,
              visibility: current.visibility,
              temp: current.main.temp,
              feels_like: current.main.feels_like,
              temp_min: current.main.temp_min,
              temp_max: current.main.temp_max,
              pressure: current.main.pressure,
              humidity: current.main.humidity,
              lon: current.coord.lon,
              lat: current.coord.lat,

              weather_id: current.weather[0].id,
              weather_main: current.weather[0].main,
              weather_description: current.weather[0].description,
              weather_icon: current.weather[0].icon,
              icon: `http://openweathermap.org/img/wn/${current.weather[0].icon}@2x.png`
            }
            
            // var date = new Date(response.data.dt * 1000)
            // self.current_result.datetime = date.toUTCString()
            self.flag_current_weather = true

            //now load for 5 days forecast
            self.getForecast(url2)
                // this.currentTemp = response.data.main.temp;
                // this.minTemp = response.data.main.temp_min;
                // this.maxTemp = response.data.main.temp_max;
                // this.pressure = response.data.main.pressure;
                // this.humidity = response.data.main.humidity + '%';
                // this.wind = response.data.wind.speed + 'm/s';
                // this.overcast = response.data.weather[0].description;
                // this.icon = "images/" + response.data.weather[0].icon.slice(0, 2) + ".svg";
                // this.sunrise = new Date(response.data.sys.sunrise*1000).toLocaleTimeString("en-GB").slice(0,4);
                // this.sunset = new Date(response.data.sys.sunset*1000).toLocaleTimeString("en-GB").slice(0,4);
        })
        .catch(error => {
          console.log(error);
        })
    },
    getForecast(url2){
      const self = this
       axios
          .get(url2)
          .then(response => {
            console.log("forecast result ",response)
            self.forecast = []
            let data = response.data
            response.data.daily.forEach(function(item, index){
              if(index < 6){
                let obj = {
                  dt: item.dt,
                  clouds: item.clouds,
                  wind_speed: item.wind_speed,
                  temp: self.getNowTemp(item.temp),
                  temp_min: item.temp.min,
                  temp_max: item.temp.max,
                  pressure: item.pressure,
                  humidity: item.humidity,
                  lon: data.lon,
                  lat: data.lat,
                  weather_id: item.weather[0].id,
                  weather_main: item.weather[0].main,
                  weather_description: item.weather[0].description,
                  weather_icon: item.weather[0].icon,
                  icon: `http://openweathermap.org/img/wn/${item.weather[0].icon}@2x.png`,

                  active: index === 0 ? 'true' : 'false'
                  // now_weather: self.getNowTemp(item.feels_like)
                }

                self.forecast.push(obj)
              }
            })
            self.slide = 0
            let chosen = 0
            self.carousel_view = {
              dt: self.forecast[chosen].dt,
              clouds: self.forecast[chosen].clouds,
              wind_speed: self.forecast[chosen].wind_speed,
              temp: self.forecast[chosen].temp,
              temp_min: self.forecast[chosen].temp_min,
              temp_max: self.forecast[chosen].temp_max,
              pressure: self.forecast[chosen].pressure,
              humidity: self.forecast[chosen].humidity,
              lon: self.forecast[chosen].lon,
              lat: self.forecast[chosen].lat,
              weather_id: self.forecast[chosen].weather_id,
              weather_main: self.forecast[chosen].weather_main,
              weather_description: self.forecast[chosen].weather_description,
              weather_icon: self.forecast[chosen].weather_icon,
              icon: self.forecast[chosen].icon
            }
            self.flag_carousel = true
        })
        .catch(error => {
          console.log(error);
        })
    }
  },
  created () {

  },
  mounted () {
    const self = this
    let temp = JSON.parse(localStorage.getItem("city_list") || "[]")
    if(temp.length !== 0){
      this.cityList = temp
    }

    let favourite = JSON.parse(localStorage.getItem("favourite") || "[]")
    if(favourite.length !== 0){
      this.favourite = favourite
    }
  }
}
</script>
