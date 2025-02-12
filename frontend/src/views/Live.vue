<template>
    <v-container fluid align = "center" class = "container0">
        <VRow max-width = "1200">
            <VCol cols = "9">
                <figure class = "highcharts-figure">
                    <div id = "container">
                        <canvas id = "chart"></canvas>
                    </div>
                </figure>
            </VCol>

            <VCol cols = "3">
                <v-card class = "card" color = "primaryContainer" subtitle = "Temperature">
                    <v-card-item>
                        <span text-h3 class = "text-onPrimaryContainer">{{ temperature }}</span>
                    </v-card-item>
                </v-card>

                <v-card class = "card" color = "tertiaryContainer" subtitle = "Heat Index (Feels Like)">
                    <v-card-item>
                        <span text-h3 class = "text-onTertiaryContainer">{{ heatindex }}</span>
                    </v-card-item>
                </v-card>

                <v-card class = "card" color = "secondaryCOntainer" subtitle = "Humidity">
                    <v-card-item>
                        <span text-h3 class = "text-onSecondaryContainer">{{ humidity }}</span>
                    </v-card-item>
                </v-card>
            </VCol>
        </VRow>

        <VRow max-width = "1200" justify = "start">
            <VCol cols = "9">
                <figure class = "highcharts-figure">
                    <div class = "container1">
                        <canvas id = "chart1"></canvas>
                    </div>
                </figure>
            </VCol>
        </VRow>
    </v-container>
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
import { useAppStore } from "@/store/appStore"; 
import {storeToRefs} from "pinia";
const Mqtt = useMqttStore();
const { payload, payloadTopic } = storeToRefs(Mqtt); 

import Highcharts from 'highcharts';
import more from 'highcharts/highcharts-more';
import Exporting from 'highcharts/modules/exporting';
Exporting(Highcharts);
more(Highcharts);
 
 
// VARIABLES
const router = useRouter();
const route = useRoute();  
const tempHiChart = ref(null); // Chart object
const humidHiChart = ref(null); // Chart object
const points = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously.
const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph.

// COMPUTED PROPERTIES
const temperature = computed(()=>{
if(!!payload.value){
return `${payload.value.temperature.toFixed(2)} °C`;
}
});

const heatindex = computed(()=>{
if(!!payload.value){
return `${payload.value.heatindex.toFixed(2)} °C`;
}
});

const humidity = computed(()=>{
if(!!payload.value){
return `${payload.value.humidity.toFixed(2)} °C`;
}
});

// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    CreateCharts();
    Mqtt.connect(); // Connect to Broker located on the backend
    setTimeout( ()=>{
        // Subscribe to each topic
        Mqtt.subscribe("620162688");
        //Mqtt.subscribe("topic2");
    },3000);

    /*const ctx = document.querySelector('#chart'); // Select canvas for rendering chart
    chart = new Chart(ctx, config ); // create chart

    const ctx1 = document.querySelector('#chart1'); // Select canvas for rendering chart
    chart1 = new Chart(ctx, config1 ); // create chart*/
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
});

const CreateCharts = async () => {
// TEMPERATURE CHART
tempHiChart.value = Highcharts.chart('container', {
    chart: { zoomType: 'x' },
    title: { text: 'Air Temperature and Heat Index Analysis', align: 'left' },
    yAxis: {
        title: { text: 'Air Temperature & Heat Index' , style:{color:'#000000'}},
        labels: { format: '{value} °C' }
    },

    xAxis: {
        type: 'datetime',
        title: { text: 'Time', style:{color:'#000000'} },
    },

    tooltip: { shared:true, },

    series: [
        {
            name: 'Temperature',
            type: 'spline',
            data: [],
            turboThreshold: 0,
            color: Highcharts.getOptions().colors[0]
        },
        {
            name: 'Heat Index',
            type: 'spline',
            data: [],
            turboThreshold: 0,
            color: Highcharts.getOptions().colors[1]
    } ],
});

humidHiChart.value = Highcharts.chart('container1', {
    chart: { zoomType: 'x' },
    title: { text: 'Humidity Analysis (Live)', align: 'left' },
    yAxis: {
        title: { text: 'Air Temperature & Heat Index' , style:{color:'#000000'}},
        labels: { format: '{value} %' }
    },

    xAxis: {
        type: 'datetime',
        title: { text: 'Time', style:{color:'#000000'} },
    },

    tooltip: { shared:true, },

    series: [
        {
            name: 'Temperature',
            type: 'spline',
            data: [],
            turboThreshold: 0,
            color: Highcharts.getOptions().colors[0]
        },
        {
            name: 'Heat Index',
            type: 'spline',
            data: [],
            turboThreshold: 0,
            color: Highcharts.getOptions().colors[1]
    } ],
});
};

// WATCHERS
watch(payload,(data)=> {
if(points.value > 0){ points.value -- ; }
else{ shift.value = true; }
tempHiChart.value.series[0].addPoint({y:parseFloat(data.temperature.toFixed(2)) ,x: data.timestamp * 1000 }, true, shift.value);
tempHiChart.value.series[1].addPoint({y:parseFloat(data.heatindex.toFixed(2)) ,x: data.timestamp * 1000 }, true, shift.value);
humidHiChart.value.series[0].addPoint({y:parseFloat(data.humidity.toFixed(2)) ,x: data.timestamp * 1000 }, true, shift.value);
})

</script>


<style scoped>
/** CSS STYLE HERE */
.card{
    margin-bottom: 5;
    max-width: 500;
}

Figure{
    border: 2px solid black;
}

container0.container{
    background-color: surface;
}
</style>
  