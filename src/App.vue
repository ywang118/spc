<template>
  <v-app>
    <v-main>
      <!-- <p v-bind:style="{ color: 'red'}"> dates-test : {{dates}} {{sampleData}}</p>
      <p v-bind:style="{ color: 'red'}">average:{{aveChange}}</p>
      <p v-bind:style="{ color: 'red'}">sd: {{sdChange}}</p> -->
      <div class="info-div">
        <v-container fluid class="myGrid">
          <v-card color="orange lighten-2" tile flat
            max-width="750"
            class="orange"
          >
            <v-card-text>
              <v-list one-line>
                <v-list-item>
                  <DatePicker :defaultDates="dates" @input="data_select" @onDateRangeChange="onDateRangeChange($event)" />
                </v-list-item>
                <v-list-item>
                  <v-checkbox
                      v-model="cal_all"
                      :label="`隨時間變化（全部資料計算標準差）: ${cal_all.toString()}`"
                  />
                </v-list-item>  
                <v-list-item>
                  <v-row>
                    <v-col
                      cols="12"
                      sm="6"
                      md="3"
                    >
                      <v-text-field 
                      outlined
                        v-model="quantity"
                        label="標準差"
                        @input="data_select"
                      />
                    </v-col> 
                  </v-row>
                </v-list-item>
              </v-list>
            </v-card-text>
          </v-card>
          <v-card class="indigo" color="indigo lighten-2" dark tile flat>
            <v-card-text class="center-text">
              <p>上限： {{ucl}} </p>
              <p>下限： {{lcl}} </p>
              <p>中心線： {{aveChange}} </p>
            </v-card-text>
          </v-card>
        </v-container>
      </div>

      <!-- <div class="spc_lines">
        <v-container  fill-height >
          <v-row justify="center" align="center"  style="height: 20px;">
            <v-col cols="12" sm="4" md="4"  >
              <p>上限： {{ucl}} </p>
            </v-col>
            <v-col cols="12" sm="4" md="4">
              <p>下限： {{lcl}} </p>
            </v-col>
            <v-col cols="12" sm="4" md="4">
              <p>中心線： {{aveChange}} </p>
            </v-col>
          </v-row>
        </v-container>  
      </div> -->
      <SPC
          chartTitle="SPC"
          :chartData="ifSelected"
          :ucl="ucl"
          :lcl="lcl"
          :cl="aveChange"
      />
    </v-main>
  </v-app>
</template>

<script>
import SPC from './components/SPC.vue';
import DatePicker from './components/DatePicker.vue';

export default {
  name: 'App',

  components: {
    SPC,
    DatePicker,
  },

  data: () => ({
    chartData: {
      x: [
        "2010-01-10",
        "2010-02-10",
        "2010-03-10",
        "2010-04-10",
        "2010-05-10",
        "2010-06-10",
        "2010-07-10",
        "2010-08-10",
        "2010-09-10",
        "2010-10-10",
        "2010-11-10",
        "2010-12-10",
        "2011-01-10",
        "2011-02-10",
        "2011-03-10",
        "2011-04-10",
        "2011-05-10",
        "2011-06-10",
        "2011-07-10",
        "2011-08-10",
      ], // 時間序列
      y: [20,22,21,19,5,18,20,22,17,19,
        23,23,22,17,19,20,19,22,18,23] //defect 數量
    },
    // //Q3
    // ucl: 27.25,
    // //Q1
    // lcl: 12.75,
    // 標準差
    quantity: 2,
    // 計算標準差: True = 算全部；False = 根據選擇日期區間
    cal_all: false,
    // 計算標準差，只根據此區間計算
    dates: ['2010-01-10', '2011-08-10'],
    //選擇日期
    sampleData: {}

  }),      //--------end of data                 -------------------------//
  computed:{

    //中位数
    median: function(){
      return (arr)=> {
        //  const arr = this.chartData.y
        // if(arr.length ===0) return 0;
        const values = [].slice.call(arr).sort(function(a,b){
          return a-b;
        });

        var half = Math.floor(values.length / 2);

        if (values.length % 2)
          return values[half];
        return (values[half - 1] + values[half]) / 2.0;
      }
    },
    //average
    average: function(){
      return (values)=> {
        const ave = (array) => array.reduce((a, b) => a + b) / array.length
        return ave(values)
      }
    },
    //标准差
    sd1: function(){
      return (values)=>{
        const n = values.length
        const mean = values.reduce((a, b) => a + b) / n
        return Math.sqrt(values.map(x => Math.pow(x - mean, 2)).reduce((a, b) => a + b) / (n-1))
      }
    },
    //四舍五入
    round: function() {
      return (value,decimals)=> {
        return Number(Math.round(value +'e'+ decimals) +'e-'+ decimals).toFixed(decimals);
      }
    },
    //ucl
    ucl: function(){
      return parseFloat(this.round(this.aveChange + this.quantity * this.sdChange,2))
    },
    //lcl
    lcl: function(){
      return parseFloat(this.round(this.aveChange- this.quantity * this.sdChange,2))
    },
    //change date format
    dateFormat: function(){
      return (str)=> {
        return str.split('-').map(x=>+x)
      }
    },
    //filter date
    resultDate(){
      const startDate = new Date(this.dateFormat(this.dates[0]))
      const endDate = new Date(this.dateFormat(this.dates[1]))
      const result = this.chartData.x.filter((dateStr)=>{
         var date = new Date(this.dateFormat(dateStr))
        return date >= startDate && date <= endDate
      })
      return result
    },
    // find out all the indexes that is within the selected date range
    mapDate(){
      const startDate = new Date(this.dateFormat(this.dates[0]))
      const endDate = new Date(this.dateFormat(this.dates[1]))
      const result = this.chartData.x.map((dateStr)=>{
         var date = new Date(this.dateFormat(dateStr))
        return date >= startDate && date <= endDate
      })
      return result
    },
    getAllIndexes:function(){
      return (arr,val)=>{
        var indexes = [], i = -1;
        while ((i = arr.indexOf(val, i+1)) !== -1){
          indexes.push(i);
        }
       return indexes;
      }
    },
    //过滤过的data
    resultData(){
      const arr = this.getAllIndexes(this.mapDate, true)
      // console.log("arr",arr)
      let xArr = []
      let yArr = []
      let newArr = {}
      for (let i=arr[0]; i <= arr[arr.length-1]; i++){
        xArr.push(this.chartData.x[i]),
        yArr.push(this.chartData.y[i])
      }
      newArr.x = xArr
      newArr.y = yArr
      return newArr
    },

    //  *------- SPC chart  -----*

    ifSelected(){
      return Object.keys(this.sampleData).length === 0? this.chartData : this.sampleData
    },
    medianChange(){
      return Object.keys(this.sampleData).length === 0 ? this.median(this.chartData.y) : this.median(this.sampleData.y)
    },
    sdChange(){
      return this.cal_all ? this.sd1(this.chartData.y): this.sd1(this.resultData.y)
    },
    aveChange(){
      return Object.keys(this.sampleData).length === 0 ? this.average(this.chartData.y): this.average(this.sampleData.y)
    },
  }, // --------end of computed        -----------------------------------//
  methods: {
    data_select() {
      // 根據常態分布的經驗法則，有 99.7% 的觀測值落在母體平均值正負 3 個母體標準差的範圍內。
      // 因此，假設收集到的資料符合常態分佈的假設，若觀測值落在樣本平均值正負 3 個樣本標準差以外的數值，可判定該觀測值屬於離群值。
      console.log("根據時間範圍內之數據, 計算出 n 個標準差的SPC上下限", this.dates)
      // this.$emit('input', this.quantity)

      this.quantity==='' ? this.quantity=0: this.quantity=parseInt(this.quantity)
    },
    onDateRangeChange(e){
      console.log(this.dates,"dates")
      this.dates = e
       console.log(this.dates,"updated_dates")
      console.log(this.resultData, 'DATESELECTED')
      this.sampleData = this.resultData
    },
  },
};
</script>
<style>
  body {
    padding-right: 50px;
    padding-left:50px;
  }
  .myGrid {
    display: grid;
    grid-template-columns: 2fr 1fr;
    grid-template-rows:auto ;
    grid-template-areas: "orange indigo";
    grid-gap: 0.5rem;
  }
  .center-text{
    text-align: center; 
  }
  .center-text p {
    font-size:20px ;
  }
  .indigo{
    display: flex;
    justify-content: center;
    align-items: center;
    height: auto;
  }
</style>