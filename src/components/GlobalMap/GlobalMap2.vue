<template>
<div>


<div>
    <b-tabs
        active-nav-item-class="font-weight-bold text-dark" justified        
    >
        <b-tab title-link-class="text-dark" title="Primary" active @click="showNewMap(0)"></b-tab>
        <b-tab title-link-class="text-dark" title="Lower Secondary" @click="showNewMap(1)"></b-tab>
        <b-tab title-link-class="text-dark" title="Upper Secondary" @click="showNewMap(2)"></b-tab>
        <b-tab title-link-class="text-dark" title="Tertiary" @click="showNewMap(3)"></b-tab>
    </b-tabs>
</div>

<div>
    <div id="map2"></div>

     <!-- Time Slider -->
    <div id='console2'>
        <br>
        <label class="control-label2">Year&nbsp; :&nbsp; </label>
        <input type="text"  id="yearCount2" readonly>
        <br>
        <p>
        2014&nbsp; <input id="slider2" type="range" min="0" max="4" step="1" value="0" />&nbsp; 2018
        </p>
    </div>

    <!-- Legend -->
    <div class='legend-container2'>
        <div class='legend2' id='legend2' >
            <h2 class="legend2">Gender Parity Index</h2> 
            <hr/>
            
            <!-- Div where the dynamic legend is created  -->	
            <div class='legend2' id='cd-legend2' >
            </div>
        
        </div>
    </div>
</div>


</div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import JQuery from 'jquery';

let $ = JQuery;
let years = [2014,2015,2016,2017,2018];
let add_legend = 1;
let popup = new mapboxgl.Popup({
                        closeButton: false,
                        closeOnClick: false
                    });

export default {
    data: () => ({
      accessToken: 'pk.eyJ1Ijoic2hlcnJ5anljIiwiYSI6ImNqb2pteTAzdjA2YmszdXBqanZ2YmNlM2wifQ.2_9XWJxI8fDvh4d_hLlrWA',
      // store url of sources of differnet education level (GER)
      ger_source_url: ['mapbox://haojun9612.100riih2','mapbox://haojun9612.9w8thqa7','mapbox://haojun9612.45tr0mwb','mapbox://haojun9612.02nt1p5j'],
      // store names of source layer of differnet education level (GER)
      ger_sourcelayer: ['GPIPrimarySmallV-d86rx5','GPILowersecondarySmallV-6ltjez','GPIUppersecondarySmallV-5co3dz','GPITertinarySmallV-3t8be0'],
    }),
    created(){
        this.map = null;
        add_legend = 1;
    },
    methods:{
        showNewMap(idx){
            this.addContent(this.map,idx);
        },
        // change map content for different education levels
        addContent(map,idx){
            var current_idx = $( "#slider2" ).val();
            var set_year = years[current_idx];
            var data_url = this.ger_source_url[idx];
            var sourcelayer = this.ger_sourcelayer[idx];
            map = new mapboxgl.Map({
                container: 'map2',
                style: 'mapbox://styles/mapbox/light-v10',
                zoom: 1.5,
                maxZoom: 2.5,
                center: [20, 15],
            });
        
            // arrays that will be used to style the layer division 
            var breaks = [
                                [0.00001, 'rgb(255,255,102)'],
                                [0.97, 'rgb(204,153,48)'],
                                [1.03, 'rgb(204,0,0)'],
            ];
            
            // arrays that will be used to populate the legend 
            var legendLabels2 = [
                        // Label text that will appear in the legend [0]
                                ['< 0.97'],
                                ['0.97 - 1.03'],
                                ['> 1.03'],                        
            ];	
            var breaksRev = breaks.slice().reverse();
            var legendLabelsRev2 = legendLabels2.slice().reverse();
            var legend2 = document.getElementById('cd-legend2'); 
    
            map.on('load', function() {
                    // show map of primary level for default
                    map.addSource('GERprimary', {
                        'type': 'vector',
                        'url': data_url,
                        'minzoom': 0,
                    });
                    
                    map.addLayer({
                        'id': 'ger-primary',
                        'minzoom': 0,
                        'maxzoom': 10,
                        'type': 'fill',
                        'source': 'GERprimary',
                        'source-layer': sourcelayer,
                        'filter': ['==', 'Time', set_year], 
                        'paint': {
                            'fill-color': {
                                // Set polygon fill color based on attribute for population density CDPOPDENS
                                property: 'Value',
                                type : 'interval',
                                // Loading array declared in the breaks variable 
                                stops: breaks,
                                },
                            'fill-opacity': 0.6
                        }    
                    });            

                    // When the user click the layer, show information of country
                    map.on('click', 'ger-primary', function(e) {
                        if (e.features.length > 0) {
                            var propObj = e.features[0].properties;
                            var line1 = '<strong>'+propObj.CNTRY_NAME+'</strong><br/>';
                            var line2 = '<p>'+propObj.Value.toFixed(2)+' % </p>';
                            popup.remove();
                            // show popup
                            popup
                            .setLngLat(e.lngLat)
                            .setHTML(line1+line2)
                            .addTo(map);
                        }
                    });
                            
                    map.on('mouseleave', 'ger-primary', function() {
                        map.getCanvas().style.cursor = '';

                    });
                    // Change the cursor to a pointer when the mouse is over a country
                    map.on('mouseenter', 'ger-primary', function() {
                        map.getCanvas().style.cursor = 'pointer';
                    });
                    // add legends during initial stage
                    if (add_legend == 1){
                        breaksRev.forEach(function(layer, i){
                    
                            // Creating elements to fill the legend div with id = 'cd-legend' 
                            // each stop gets a 'row' 
                            var item = document.createElement('div');
                            //add a 'key' to the row. A key will be the color key
                            var key = document.createElement('span');
                            //add a value variable to the 'row' in the legend
                            var value = document.createElement('span');
                            
                            //the key will take on the shape and style properties defined here, in the HTML
                            key.className = 'legend-key2'; 
                            // the background color is retreived from the breaks array
                            key.style.backgroundColor = layer[1]; 

                            // give the value variable a placeholder id that we can access and update with custom labels
                            value.id = 'legend-value2-' + i;
                            
                            //add the key (color key) to the legend row
                            item.appendChild(key); 
                            //add the placeholder value to the legend row
                            item.appendChild(value);
                            // Add row to the legend
                            legend2.appendChild(item);        
                        });
                
                        legendLabelsRev2.forEach(function(layer, i){
                            //as we iterate through the arrays get the correct row, and add the appropriate text 
                            document.getElementById('legend-value2-' + i).textContent = layer[0];             
                        });  
                        add_legend = 0;
                    }
            }); 
            // set year for time slider
            $( "#yearCount2" ).val(set_year);
            $( "#slider2" ).val(current_idx);

            $( "#slider2" ).change(function(e) {
                    popup.remove();
                    var year = parseInt(e.target.value, 5);
                    var filters = ['==', 'Time', years[year]];
                    // runs filter query on Mapbox layer based on the location/value of the handle on the slider
                    map.setFilter('ger-primary', filters);
                    // update the year counter based on the location/value of the handle on the slider
                    $( "#yearCount2" ).val(years[year]);
                });

            var items =['2014','2015','2016','2017','2018'];

                // Calculate the legth of the legend based on number of items declared to identify spacing required
                // Do not change when updating the items variable
            var length = 100 / (items.length - 1);
            $.each(items, function(key,value){
                
                var spacing = length;
                if(key === 0 || key === items.length-1)
                spacing = length/2;
                    
                    // Updates sliderLegend div below the slider using spacing variable to set calculated label spacing width and include label value for the label
                    // Do not change when updating the items variable
                $("#sliderLegend").append("<label_scale style='width: "+spacing+"%'>"+value+"</label_scale>");
            });

        },

    },

    mounted(){
         mapboxgl.accessToken = 'pk.eyJ1Ijoic2hlcnJ5anljIiwiYSI6ImNrYWhuNnUyaDBpMW8yeHQ5YmU5bjRxbmYifQ.rTKiRvlmkUa2IfJl9ToD9g';

        // initialize map content with primary education data
        var init_idx = 0;
        this.addContent(this.map, init_idx);
        
    }
}
        
</script>

<style>
#map2 { position: absolute; top: 100px; bottom: 0; width: 96%; height:80%;}

    .map-overlay {
        font: 12px/20px 'Helvetica Neue', Arial, Helvetica, sans-serif;
        position: absolute;
        width: 25%;
        top: 0;
        left: 0;
        padding: 10px;
    }

    .map-overlay .map-overlay-inner {
        background-color: #fff;
        box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
        border-radius: 3px;
        padding: 10px;
        margin-bottom: 10px;
    }

    .map-overlay h2 {
        line-height: 24px;
        display: block;
        margin: 0 0 10px;
    }

    .map-overlay .legend2 .bar {
        height: 10px;
        width: 100%;
        background: linear-gradient(to right, #fca107, #7f3121);
    }

    .map-overlay input {
        background-color: transparent;
        display: inline-block;
        width: 100%;
        position: relative;
        margin: 0;
        cursor: ew-resize;
    }
    
    /*Container bottom left*/
    #console2 {
        position: absolute;
        width: 420px;
        margin: 10px;
        margin-bottom: 3%;
        padding: 10px 20px;
        background-color: white;
        bottom:20px;
        z-index:2;
        opacity: 0.7;
    }

    #console2 hr {
        margin-left:-20px; 
        margin-right:-20px;  
        color: #123455; 
        height: 1px; 
        border: 0; 
        border-top: 1px solid #ccc; 
        padding: 0; 
    }

    /*Mapbox Legend */
    .legend2 {
        font: 12px/20px 'Helvetica Neue', Arial, Helvetica, sans-serif;
        background-color: white; 
        padding: 10px;
        opacity: 0.9;
    }

    #cd-legend2 {
        text-align: left;
    }
    
    /* legend test */
    .legend-container2 {
        position: absolute;
        margin: 10px;
        top: 100px;
        left: 10px;
        padding: 0px 10px;
        margin-bottom: 30px;
        z-index: 1;
        max-width: 250px
    }
    
    /* Legend title */
    .legend2 h2 {
        padding-top: 8px;
        margin:0;
        text-align: center;
    }

    .legend2 hr {
        color: #123455; 
        border: 0; 
        border-top: 1px solid #ccc; 
        padding: 0; 
        margin-top:5px;
        margin-bottom:5px
    }
        
    .legend-key2 {
        display: inline-block;
        /*size of color key*/
        width: 30px; 
        height: 15px;
        /*position of color key*/
        margin-right: 15px; 
        margin-left: 10px; 
    }  
    
    /* Customize jQuery Slider UI - Handle color and shape*/  
    .ui-slider .ui-slider-handle {
        width: 20px;
        height: 20px;
        background: #f6931f;
        border-radius: 50%;
        border: none;
        cursor: pointer;
    }

    /* Customizing Slider UI - Handle placement along slider to line up*/ 
    .ui-slider-horizontal .ui-slider-handle {
        top: 50%;
        margin-top: -10px;
    }

    .ui-slider-horizontal {
        height: 6px;
    }

    /* Style the counter*/
    #yearCount2{
        border:0; 
        color:#f6931f; 
        font-weight:bold; 
        font-size:14px;
        width:50px
    }

    #console2 > label {
        font-weight:bold; 
        font-size:14px;
    }

</style>