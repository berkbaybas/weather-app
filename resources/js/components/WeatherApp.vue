<template>
<div class="text-white mb-8">
    <div class="places-input text-gray-800">
        <input type="search" id="address" class="form-control" placeholder="In which city do you live" />
        <p>Selected: <strong id="address-value">none</strong></p>
    </div>
    <div class="weather-container font-sans w-128 max-w-lg rounded-lg overflow-hidden bg-gray-900 shadow-lg mt-4">
        <div class="current-weather flex items-center justify-between px-6 py-8">
            <div class="flex items-center">
                <div>
                    <div class="text-6xl font-semibold">{{ currentTemperature.actual }}°C</div>
                    <div>{{ currentTemperature.feels }}°C</div>
                </div>
                <div class="mx-5">
                    <div class="font-semibold">
                        {{ currentTemperature.summary }}
                    </div>
                    <div>{{ location.name }}</div>
                </div>
            </div>
            <div>
                <canvas ref="iconCurrent" id="iconCurrent" width="96" height="96"></canvas>
            </div>
        </div><!-- end current-weather-->
        <div class="future-weather text-sm bg-gray-800 px-6 py-8 overflow-hidden">
            <div v-for="(day, index) in daily" :key="index" class="flex items-center" :class="{'mt-8' : index > 0}" v-if="index < 5">
                <div class="w-1/6 text-lg text-gray-200">{{ toDayOfWeek(day.time)}}</div>
                <div class="w-4/6 px-4 flex items-center">
                    <div>
                        <canvas :id="`icon${index+1}`" :data-icon="toKebabCase(day.icon)" width="24" height="24"></canvas>
                    </div>
                    <div class="ml-3">{{ day.summary}}</div>
                </div>
                <div class="w-1/6 text-xs text-right">
                    <div>{{Math.round(day.temperatureHigh)}}°C</div>
                    <div>{{Math.round(day.temperatureLow)}}°C</div>
                </div>
            </div>

        </div><!-- end future-weather-->
    </div> <!-- end weather-container-->
</div>
</template>

<script>
export default {
    data() {
        return {
            currentTemperature: {
                actual: '',
                feels: '',
                summary: '',
                icon: '',
            },
            daily: [],
            location: {
                name: 'Bursa, Türkiye',
                lat: 43.6532,
                lng: -79.38323
            }
        }
    },
    watch : {
        location : {
            handler(newValue, oldValue){
                this.fetchData()
            },
            deep : true
        }
    },
    mounted() {
        this.fetchData()
        var placesAutocomplete = places({
            appId: 'plM0IEBQNOUD',
            apiKey: '9bcd949814588ad09167cee00390807d',
            container: document.querySelector('#address')
            }).configure({
                type: 'city',
                aroundLatLngViaIP: false,
            })
        

        var $address = document.querySelector('#address-value')

        placesAutocomplete.on('change',(e) => {
            $address.textContent = e.suggestion.value

            this.location.name = `${e.suggestion.name}, ${e.suggestion.country}`
            this.location.lat = e.suggestion.latlng.lat
            this.location.lng = e.suggestion.latlng.lng
        })

        placesAutocomplete.on('clear', () => {
            $address.textContent = 'none'
        })
    },
    methods: {

        fetchData() {
            var skycons = new Skycons({
                "color": "white"
            });
            fetch(`/api/weather?lat=${this.location.lat}&lng=${this.location.lng}`)
                .then(response => response.json())
                .then(data => {
                    console.log(data);
                    this.currentTemperature.actual = Math.round(data.currently.temperature)
                    this.currentTemperature.feels = Math.round(data.currently.apparentTemperature)
                    this.currentTemperature.summary = data.currently.summary
                    this.currentTemperature.icon = this.toKebabCase(data.currently.icon)

                    this.daily = data.daily.data

                    skycons.add('iconCurrent', this.currentTemperature.icon)
                    skycons.play()

                    this.$nextTick(() => {
                        skycons.add('icon1', document.getElementById('icon1').getAttribute('data-icon'))
                        skycons.add('icon2', document.getElementById('icon2').getAttribute('data-icon'))
                        skycons.add('icon3', document.getElementById('icon3').getAttribute('data-icon'))
                        skycons.add('icon4', document.getElementById('icon4').getAttribute('data-icon'))
                        skycons.add('icon5', document.getElementById('icon5').getAttribute('data-icon'))

                        skycons.play()
                    })
                })
        },
        toKebabCase(stringToConvert) {
            return stringToConvert.split(' ').join('-')
        },
        toDayOfWeek(timestamp) {
            const newDate = new Date(timestamp * 1000)
            const days = ['PAZ', 'PZT', 'SAL', 'ÇAR', 'PRŞ', 'CUM', 'CMR']

            return days[newDate.getDay()]
        }
    }
}
</script>

<style scope>
.ap-suggestion-icon svg {
    @apply inline-block;
    @apply -mt-3;
}
</style>