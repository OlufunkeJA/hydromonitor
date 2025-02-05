<template>
    <!-- HTML HERE -->    
    <v-container fluid align = "center">
        <v-row max-width = "1200" justify = "center">
            <v-column justify = "center">
                <v-sheets class = "sheets, rounded-t-lg">
                    <v-card class = "text-secondary" title = "LED Controls" color = "surface" subtitle = "Recent Settings" variant = "tonal" flat></v-card>
                </v-sheets>

                <v-sheets class = "sheets">
                    <v-card class = "card2" color = "surface" variant = "tonal">
                        <v-slider class = "pt-2 bg-surface" append-icon = "mdi:mdi-car-light-high" density = "compact" thumb-size = "16" color = "secondary" label = "Brightness" direction = "horizontal" min = "0" max = "250" step = "10" show-ticks thumb-label = "always" v-model="led.brightness"></v-slider>
                    </v-card>
                </v-sheets>

                <v-sheets class = "sheets" justify = "center" align = "center">
                    <v-card class = "pt-5">
                        <v-slider class = "pt-2 bg-surface" append-icon = "mdi:mdi-led-on" density = "compact" thumb-size = "16" color = "secondary" label = "LED Nodes" direction = "horizontal" min = "1" max = "7" step = "1" show-ticks thumb-label = "always" v-model="led.leds"></v-slider>
                    </v-card>
                </v-sheets>

                <v-sheets class = "sheets, pa-2" justify = "center" align = "center">
                    <v-progress-circular rotate = "0" size = "200" width = "15" :model-value="led.leds *15" :color="indicatorColor">
                        <span class="text-onSurface font-weight-bold">{{ led.leds }} LED(s)</span>
                    </v-progress-circular>
                </v-sheets>
            </v-column>

            <v-column justify = "center">
                <v-color-picker v-model="led.color"></v-color-picker>
            </v-column>
        </v-row>
    </v-container>
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
 
 
// VARIABLES
const router = useRouter();
const route = useRoute();  
const led = reactive({"brightness":255,"leds":1,"color":{ r: 255, g: 0, b: 255, a: 1 }});
let timer, ID = 1000;


// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
});

// WATCHERS
watch(led,(controls)=>{
    clearTimeout(ID);

    ID = setTimeout(()=>{
        const message = JSON.stringify({"type":"controls","brightness":controls.brightness,"leds":controls.leds,"color": controls.color});
        Mqtt.publish("620162688_sub",message); // Publish to a topic subscribed to by the hardware
    },1000)
})

// COMPUTED PROPERTIES
const indicatorColor = computed(()=>{
    return `rgba(${led.color.r},${led.color.g},${led.color.b},${led.color.a})`
})

</script>


<style scoped>
/** CSS STYLE HERE */

.sheets{
    color: surface;
    elevation: 0;
    max-width: 800;
    width: 100%;
    margin-bottom: 1;
}

.card2{
    padding-top: 5;
}

</style>
  