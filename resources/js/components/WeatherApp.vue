<template>
    <div class="text-white mb-8 mx-2 md:mx-0 w-screen overflow-hidden">
        <div class="places-input text-gray-800 mx-auto w-full md:w-128">
            <input type="text" id="address" placeholder="Where are we going?" />
            <p>Selected: <strong>{{ location.name }}</strong></p>
        </div>
        <div class="weather-container font-sans w-full md:w-128 mx-auto max-w-lg overflow-hidden bg-gray-900 shadow-lg mt-4 rounded-lg">
            <div class="current-weather flex justify-between items-center px-6 py-8">
                <div class="flex items-center">
                    <div>
                        <div class="text-4xl md:text-6xl font-semibold">{{ currentTemperature.actual }}{{ currentUnit }}</div>
                        <div class="text-xs md:text-sm ">Feels like {{ currentTemperature.feels }}{{ currentUnit }}</div>
                    </div>
                    <div class="mx-5">
                        <div class="text-xs md:text-sm font-semibold">{{ currentTemperature.summary }}</div>
                        <div class="text-xs md:text-sm">{{ location.name }}</div>
                    </div>
                </div>
                <div>
                    <canvas ref="iconCurrent" id="iconCurrent" width="84" height="84"></canvas>
                </div>
            </div>

            <div class="future-weather text-xs md:text-sm bg-gray-800 px-6 py-8 overflow-hidden">
                <div class="flex items-center" v-for="(day, index) in daily" :key="'day-' + index" :class="{ 'mt-8' : index > 0  }">
                    <div class="w-1/6 text-lg text-gray-200">{{ toDayOfWeek(day.time) }}</div>
                    <div class="w-4/6 px-4 flex items-center">
                        <div>
                            <canvas :id="`icon${index+1}`" :data-icon="toKebabCase(day.icon)" width="24" height="24"></canvas>
                        </div>
                        <div class="ml-3">{{ day.summary }}</div>
                    </div>
                    <div class="w-1/6 text-right">
                        <div>{{ Math.round(day.temperatureHigh) }}{{ currentUnit }}</div>
                        <div>{{ Math.round(day.temperatureLow) }}{{ currentUnit }}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        mounted() {
            this.fetchData();
            const placesAutocomplete = places({
                appId: 'plCL45Q26BEL',
                apiKey: '4b8569f86ae5e7abceb94261fa8726fc',
                container: document.querySelector('#address')
            }).configure({
                type: 'city',
                aroundLatLngViaIP: false
            });
            
            const $address = document.querySelector('#address-value');
            placesAutocomplete.on('change', (e) => {
                const { value, latlng, name, country } = e.suggestion;

                // $address.textContent = value;
                this.location.name = `${name}, ${country}`;
                this.location.lat = latlng.lat;
                this.location.lng = latlng.lng;
            });

            placesAutocomplete.on('clear', () => {
                // $address.textContent = 'none';
                this.location.name = `none`;
                this.location.lat = '';
                this.location.lng = '';
            });
        }, watch: {
            location: {
                handler(newValue, oldValue) {
                    this.fetchData();
                }, deep: true
            }
        }, data() {
            return {
                daysToShow: 5,
                currentUnit: '°C',
                currentTemperature: {
                    actual: '',
                    feels: '',
                    summary: '',
                    icon: ''
                }, daily: [

                ], location: {
                    name: 'Puducherry, India',
                    lat: 11.9341,
                    lng: 79.8306
                }
            }
        },methods: {
            fetchData() {
                var skycons = new Skycons({'color': 'white'});

                fetch(`/api/weather?lat=${this.location.lat}&lng=${this.location.lng}`).then(response => response.json()).then(data => {
                    const{ currently, daily } = data;
                    const { temperature, apparentTemperature, summary, icon } = currently;

                    this.currentTemperature.actual = Math.round(temperature);
                    this.currentTemperature.feels = Math.round(apparentTemperature);
                    this.currentTemperature.summary = summary;
                    this.currentTemperature.icon = this.toKebabCase(icon);

                    this.daily = daily.data.slice(0, this.daysToShow);

                    skycons.add('iconCurrent', this.currentTemperature.icon);
                    skycons.play();

                    this.$nextTick(() => {
                        new Array(this.daysToShow).fill(0).map((v, i) => {
                           skycons.add(`icon${i+1}`, document.getElementById(`icon${i+1}`).getAttribute('data-icon')); 
                        });
                        skycons.play();
                    });
                });
            }, toKebabCase(stringToConvert) {
                return stringToConvert.split(' ').join('-');
            }, toDayOfWeek(timestamp) {
                const newDate = new Date(timestamp * 1000);
                const days = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT'];

                return days[newDate.getDay()];
            }
        }
    }
</script>
