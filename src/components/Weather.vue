<template>
  <v-container>
    <v-responsive
      class="align-centerfill-height mx-auto"
      max-width="700"
    >
      <v-row class="mb-5">
        <v-col col="12">
          <v-btn id="nowBtn" class="btn-f mr-3" @click="fetchWeather('now',days[0])">Now</v-btn>
          <v-btn id="todayBtn" class="btn-f mr-3" @click="fetchWeather('today',days[0])">Today</v-btn>
          <v-btn 
            id="menu-activator"
            class="btn-f"
            color="default"
          >
            Select Date
          </v-btn>

          <v-menu activator="#menu-activator">
            <v-list>
              <v-list-item
                v-for="(day, index) in days"
                :key="index"
                :value="index"
              >
                <v-list-item-title @click='fetchWeather("days",day)'>{{ day }}</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-menu>            
        </v-col>
      </v-row>

      <v-row class="d-flex justify-space-between"> 
        <v-col>
          <h1>{{ weather.temp }} &#8451;</h1>
          <span>{{weather.main}} - {{ weather.description }}</span>
        </v-col>
        <v-col class="text-right">
          <img class="wIcon" :src="weather.icon" alt="weather icon">
        </v-col>
      </v-row>
      <hr class="hrStyle">
      <v-row class="mt-5">
        <v-col class="box-w mx-3">
          <h3>{{ weather.feels_like }} &#8451;</h3>
          <span>Feels like</span>
        </v-col>
        <v-col class="box-w mx-3">
          <h3>{{ weather.wind_speed }} <small>m/s</small></h3>
          <span>Wind</span>
        </v-col>
        <v-col class="box-w mx-3">
          <h3>{{ weather.wind_gust }} <small>m/s</small></h3>
          <span>Wind Gust</span>
        </v-col>
      </v-row>

      <v-row class="mt-7">
        <v-col class="box-w mx-3">
          <h3>{{ weather.wind_deg }} &#176;</h3>
          <span>Wind Deg</span>
        </v-col>
        <v-col class="box-w mx-3">
          <h3>{{ weather.humidity }} <small>%</small></h3>
          <span>Humidity</span>
        </v-col>
        <v-col class="box-w mx-3">
          <h3>{{ weather.pressure }} <small>hPa</small></h3>
          <span>Pressure</span>
        </v-col>
      </v-row>
      
      <v-row>
        <v-col class="mt-10">
          <hr class="hrStyleLarge">
        </v-col>
        <v-col cols="12">
          <!-- chart -->
          <h4>Weekly Variation</h4>
          <h5 class="mt-6">Temperature</h5>
          <div id="chart">
            <apexchart ref="realtimeChart" type="line" height="450" :options="chartOptions" :series="series"></apexchart>
          </div>
        </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script>
import VueApexCharts from "vue3-apexcharts";
  export default {
    components: {
      apexchart: VueApexCharts, 
  },
  data() {
    return {
      // chart
      series: [{
          name: "Desktops",
          data: []
      }],
      chartOptions: {
        chart: {
          height: 450,
          type: 'line',
          zoom: {
            enabled: false
          },
          toolbar: {
            show: false
          },
        },
        dataLabels: {
          enabled: true
        },
        stroke: {
          curve: 'smooth'
        },
        title: {
          text: '',
          align: 'left'
        },
        grid: {
          row: {
            colors: ['#fff', '#fff', '#fff', '#fff', '#fff'],
            opacity: 1
          },
        },
        xaxis: {
          categories: [],
        },
        tooltip:{
          enabled:false,
        }
      },
      // end chart
      days:['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday'],
      weather:{
        temp:"-",        
        feels_like:"-",                
        pressure:"-",        
        humidity:"-",        
        wind_speed:"-",        
        wind_gust:"-",        
        wind_deg:"-",
        description:"-",
        main:"-",
        icon:"https://openweathermap.org/img/w/03d.png"        
      }
    }
  },
  methods: {
    calcDays(){
      // calculate days based on today
      const d = new Date().getDay()
      const startDays = this.days.slice(d)
      const endDays = this.days.splice(0,d)
      this.days = [...startDays, ...endDays].slice(0, -1)
    },
    activeClassF(elem){
      document.querySelector('#' + elem).classList.add('active')
      document.querySelectorAll('.btn-f:not(#'+elem+')').forEach(elem => {
        elem.classList.remove('active')
      })
    },
    weatherInfos(elem){
      this.weather.pressure = (elem.main?.pressure / 1000).toFixed(2) ?? "-"
      this.weather.humidity = elem.main?.humidity ?? "-"
      this.weather.wind_speed = elem.wind?.speed ?? "-"
      this.weather.wind_gust = elem.wind?.gust ?? "-"
      this.weather.wind_deg = elem.wind?.deg ?? "-"
      this.weather.description = elem.weather[0]?.description ?? "-"
      this.weather.main = elem.weather[0]?.main ?? "-"
      this.weather.icon = "http://openweathermap.org/img/w/" + elem.weather[0]?.icon + ".png" ?? null
    },
    calcMaxTemps(dates,list){
      // empty temp arrays
      this.series[0].data=[]
      const maxTemp1 = [],maxTemp2 = [],maxTemp3 = [],maxTemp4 = [],maxTemp5 = [],maxTemp6 = [];
      // fill temp arrays for each day
      list.forEach((element) => {
        const check = element.dt_txt.substring(8,10);
        const e = element.main.temp_max
        if(check.includes(dates[0])){               
          maxTemp1.push(e)
        }
        if(check.includes(dates[1])){               
          maxTemp2.push(e)
        }
        if(check.includes(dates[2])){               
          maxTemp3.push(e)
        }
        if(check.includes(dates[3])){               
          maxTemp4.push(e)
        }
        if(check.includes(dates[4])){               
          maxTemp5.push(e)
        }
        if(check.includes(dates[5])){               
          maxTemp6.push(e)
        }
      });       
      // create yaxis data of chart
      const t1 = maxTemp1.length > 0 ? Math.max(...maxTemp1) : null
      const t2 = maxTemp2.length > 0 ? Math.max(...maxTemp2) : null
      const t3 = maxTemp3.length > 0 ? Math.max(...maxTemp3) : null
      const t4 = maxTemp4.length > 0 ? Math.max(...maxTemp4) : null
      const t5 = maxTemp5.length > 0 ? Math.max(...maxTemp5) : null
      const t6 = maxTemp6.length > 0 ? Math.max(...maxTemp6) : null
      this.series[0].data.push(t1,t2,t3,t4,t5,t6)
    },
    updatexAxis(dates,list){
      const dayMonth = []
      list.forEach(e => {
        dates.push(e.dt_txt.substring(8,10))
        dayMonth.push(e.dt_txt.substring(5,11).replace('-','/'))
      });
      this.$refs.realtimeChart.updateOptions({
        xaxis: {
          categories: [...new Set(dayMonth)] 
        }
      })
    },
    fetchWeather(value,event){
      const dates = []
      const key = '40d3d46eac369ee07e5556724b9a68f7';
      const url = `http://api.openweathermap.org/data/2.5/forecast?id=734077&onecall?lat=40.58725980318928&lon=22.948223362612612&exclude=hourly,minutely&appid=${key}&units=metric`;
      fetch(url)
        .then(response => response.json())
        .then(data => {
          // update chart x axis
          this.updatexAxis(dates,data.list);
          if (value == 'now') {
            this.activeClassF('nowBtn')
            this.weather.temp = data?.list[0]?.main?.temp ?? "-"
            this.weather.feels_like = data?.list[0]?.main?.feels_like ?? "-"
            this.weatherInfos(data?.list[0])
            this.calcMaxTemps([...new Set(dates)],data?.list)
          }
          if (value == 'today' || value == 'days') {
            const uniq = [...new Set(dates)]
            const results = [];
            const dateIndex = uniq[this.days.indexOf(event)]
            let mainTempSum = 0;
            let feelsLike = 0;
            data.list.forEach(element => {                
              if(element.dt_txt.substring(8,10).includes(dateIndex)){               
                results.push(element)
                mainTempSum += element.main.temp
                feelsLike += element.main.feels_like
              }
            });
            this.weatherInfos(results[0]) 
            this.weather.temp = (mainTempSum / results.length).toFixed(2)
            this.weather.feels_like = (feelsLike / results.length).toFixed(2)
          }
          if(value == 'today'){
            this.activeClassF('todayBtn')
          }
          if(value == 'days'){
            this.activeClassF('menu-activator')
          }
        })
        .catch(error => console.error('Error:', error));
    },
  },
  created() {
    this.calcDays();
    this.fetchWeather('now',this.days[0]);
  }
}
</script>

<style scoped>

  .wIcon{
    min-width:100px;
    object-fit: cover;
  }

  .box-w{
    border: 1px solid rgba(128, 128, 128, 0.3);
    border-radius: 8px;
  }

  .hrStyle{
    height: 4px;
    border: 0;
    box-shadow: inset 0 12px 12px -12px rgba(0, 0, 0, 0.5);
  }

  .hrStyleLarge{
    height: 20px;
    background-color: rgba(128, 128, 128, 0.20);
    border: 0;
  }

  .active{
    color:green;
    background-color: rgbA(0, 128, 0, .2);
  }

</style>
