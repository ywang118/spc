<template>
    <div class="chart">
        {{this.value}}
        {{this.pieTitle}}
        total: {{this.totalValue}}
        <div id="pie_chart" class="pie_chart" />
    </div>
</template>
<script>
import Plotly from "plotly.js-dist";
export default {
    name: "pie_chart",
    props:{
        value:{
            type:Array,
        }
    },
    data:()=>({}),
    watch: {
        value: {
            handler() {
                this.drawPlot();
            },
            deep: true
            },
    },
    methods:{
        myEventHandler() {
            this.drawPlot();
        },
        drawPlot(){
            var data = [{
                type: "pie",
                values:  this.pieValue,
                labels: this.pieLabel,
                name: 'output Proportion',
                hoverinfo: 'label+percent+name',
                hole: .4,

            }]

            var layout = {
                title: this.pieTitle,
                annotations: [
                    {
                    font: {
                        size: 20
                    },
                    showarrow: false,
                    text: this.totalValue,
                    x: 0.2,
                    y: 0.5
                    }
                    
                ],
                height: 400,
                width: 600,
               
                grid: {rows: 1, columns: 2}
                };

            Plotly.newPlot('pie_chart', data, layout)
        }
    },
    computed:{
     
        pieTitle: function(){   
            return this.value === undefined || this.value.length == 0 ? '':this.value[0].date         
        },
        pieLabel: function(){

            if  (this.value === undefined || this.value.length == 0 ){
                return []   
            } else {
                let arr = Object.keys(this.value[0])
                arr.shift()
                return arr
            }
        },
        pieValue: function(){
            if  (this.value === undefined || this.value.length == 0 ){
                return []   
            } else {
                let arr = Object.values(this.value[0])
                arr.shift()
                return arr
            }
        },
        totalValue: function(){
           if  (this.value === undefined || this.value.length == 0 ){
                return ''
            } else {
                const newArr = this.pieValue.map(v => parseInt(v, 10))
                return newArr.reduce((a, b) => a + b, 0)
            }
        }
    },
    mounted(){
    this.drawPlot();
    },
    created() {
        window.addEventListener("resize", this.myEventHandler);
    },
    destroyed() {
        window.removeEventListener("resize", this.myEventHandler);
    }
    
}
</script>