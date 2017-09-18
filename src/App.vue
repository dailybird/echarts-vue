<style>
    .echarts-container {
        width: 100%;
        height: 20rem;
    }
</style>

<template>
    <div id="app">
        <i-echart :option="option" @echarts-clicked="echartsClicked" class="echarts-container"></i-echart>
        <div style="height: 50rem;"></div>
        <i-echart :option="option" resize-signature="1234" class="echarts-container"></i-echart>
        <i-echart :option="option" class="echarts-container"></i-echart>
    </div>
</template>

<script>
    import iEchart from './components/i-echart'

    export default {
        name: 'app',
        components: {
            iEchart
        },
        data() {
            return {
                option: {
                    title: {
                        text: 'ECharts 入门示例'
                    },
                    tooltip: {},
                    legend: {
                        data: ['销量']
                    },
                    xAxis: {
                        data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
                    },
                    yAxis: {},
                    series: [{
                        name: '销量',
                        type: 'bar',
                        data: [5, 20, 36, 10, 10, 20]
                    }]
                }
            }
        },
        methods:{
            echartsClicked(params){
                console.log(params)
            }
        }
    }
</script>
