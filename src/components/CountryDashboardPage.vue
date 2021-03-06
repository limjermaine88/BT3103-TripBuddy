<template>
    <div id = "statistics_dashboard">
        <div class = "container">
            <h6>Last Refreshed: {{date}}</h6>
            <div>
                <section class = "controls">
                    <md-autocomplete :md-options = "country_options_dropdown" v-model = "selectedOption">
                        <label>Country</label>
                    </md-autocomplete>
                    <md-button @click = "updateData(selectedOption)" class = "md-raised md-primary"
                               style="padding: 8px; height: 100%; display: block; margin-left: 5px">
                        Search
                    </md-button>
                    <pin-a-country :selected-country = "selectedOption"></pin-a-country>
                </section>
                <p>
                    Using shortcut search with PINNED country list by clicking in the country tag</p>
                <b-form-group>
                    <md-chip :key = "chip" @click = "selectedOption = chip; updateData(selectedOption)" class = "md-accent"
                             md-clickable style = "padding: 5px; margin-top: 5px; margin-right: 5px"
                             v-for = "chip in userProfile.country_interested">
                        {{chip}}
                    </md-chip>
                </b-form-group>
                <section>
                    <div class = "chart controls">
                        <canvas id = "radarChart"></canvas>
                    </div>
                    <div class = "indicControls" style = "background-color:white; padding:15px; border-radius: 17px;">
                        <div>
                            <h5><b> <u> Stringency Index </u> </b></h5>
                            <p>
                                records the strictness of ‘lockdown style’ policies that primarily restrict people’s
                                behaviour </p>
                            <h5><b> <u> Government Response Index </u> </b></h5>
                            <p>
                                records how the response of governments has varied over all indicators in the database,
                                becoming stronger or weaker over the course of the outbreak </p>
                            <h5><b> <u> Containment & Health Index </u> </b></h5>
                            <p>
                                combines ‘lockdown’ restrictions and closures with measures such as testing policy and
                                contact tracing, short term investment in healthcare, as well investments in
                                vaccine </p>
                        </div>
                    </div>
                </section>
                <br> <br>
                <section style = "margin: auto">
                    <div class = "H" v-if = "this.H_indicators.length === 0">
                        <p style = "text-align: center">The Health System policies is not available for the country.</p>
                    </div>
                    <div class = "H" v-else>
                        <h6 style = "text-align: center; font-size:20px; margin-left: 30px;"><b> Health System
                                                                                                 Policies </b></h6>
                        <table id = "H_indicators">
                            <thead>
                            <tr>
                                <th>Code</th>
                                <th>Description</th>
                                <th>Value</th>
                                <th>Policy</th>
                            </tr>
                            </thead>
                            <tbody>
                            <tr v-bind:key = "row" v-for = "row in this.H_indicators">
                                <td>{{row.code}}</td>
                                <td>{{row.descr}}</td>
                                <td>{{row.value}}</td>
                                <td>{{row.policy}}</td>
                            </tr>
                            </tbody>
                        </table>
                    </div>
                    <div class = "C" v-if = "this.C_indicators.length === 0">
                        <p style = "text-align: center">The Containment and Closure Policies is not available for the
                                                        country.</p>
                    </div>
                    <div class = "C" v-else>
                        <h6 style = "text-align: center; font-size:20px; margin-left: 60px;"><b> Containment and Closure
                                                                                                 Policies </b></h6>
                        <table id = "C_indicators">
                            <thead>
                            <tr>
                                <th>Code</th>
                                <th>Description</th>
                                <th>Value</th>
                                <th>Policy</th>
                            </tr>
                            </thead>
                            <tbody>
                            <tr v-bind:key = "row" v-for = "row in this.C_indicators">
                                <td>{{row.code}}</td>
                                <td>{{row.descr}}</td>
                                <td>{{row.value}}</td>
                                <td>{{row.policy}}</td>
                            </tr>
                            </tbody>
                        </table>
                    </div>
                </section>
            </div>
        </div>
    </div>
</template>

<script>
	import {mapState} from "vuex";
	import axios from 'axios';
	import Chart from 'chart.js';
	import radarChartData from "../radar.js";
	import PinACountry from "@/components/PinACountry";
	
	export default {
		data() {
			return {
				date: 'today',
				selectedOption: '[AFG] Afghanistan',
				radarChartData: radarChartData,
				myChart: null,
				H_indicators: [],
				C_indicators: []
			}
		},
		components: {
			PinACountry
		},
		computed: {
			...mapState(['userProfile', 'country_options_dropdown'])
		},
		methods: {
			date_function: function () {
				let currentDate = new Date();
				currentDate.setDate(currentDate.getDate() - 5);
				return currentDate.toJSON().slice(0, 10);
			},
			next_date: function () {
				let currentDate = new Date();
				currentDate.setDate(currentDate.getDate() - 4);
				return currentDate.toJSON().slice(0, 10);
			},
			createChart: function (chartId, chartData) {
				const ctx = document.getElementById(chartId);
				this.myChart = new Chart(ctx, {
					type: chartData.type,
					data: chartData.data,
					optionsSortingElement: chartData.options
				});
			},
			updateData: function (countryCode) {
				const url = 'https://datasource.kapsarc.org/api/records/1.0/search/?dataset=oxford-covid-19-government-response-tracker&rows=10000&disjunctive.countryname=true&disjunctive.indicator=true&refine.indicator=Containment+Health+Index.&refine.indicator=Government+Response+Index.&refine.indicator=Stringency+Index.&q=date:%5B' + this.date_function() + 'T16:00:00Z+TO+' + this.next_date() + 'T15:59:59Z%5D&timezone=Asia/Shanghai&lang=en';
				let name = countryCode.substr(6)
				const containment = 'Containment Health Index.';
				const government = 'Government Response Index.';
				const stringency = 'Stringency Index.';
				let CI = 0.0;
				let GI = 0.0;
				let SI = 0.0;
				let code = countryCode.slice(1, 4)
				// fetch data for radar chart
				axios.get(url).then(response => {
					for (let key in response.data.records) {
						if (response.data.records[key].fields.countryname === name) {
							let indicator = response.data.records[key].fields.indicator
							if (indicator === containment) {
								CI = response.data.records[key].fields.value
							} else if (indicator === government) {
								GI = response.data.records[key].fields.value
							} else if (indicator === stringency) {
								SI = response.data.records[key].fields.value
							}
						}
					}
					// populate radar chart
					this.myChart.data.datasets[0].label = code + ' Index'
					this.myChart.data.datasets[0].data = []
					this.myChart.data.datasets[0].data.push(CI)
					this.myChart.data.datasets[0].data.push(GI)
					this.myChart.data.datasets[0].data.push(SI)
					this.myChart.update()
					// alert when no data available
					if (CI === 0.0 && GI === 0.0 && SI === 0.0) {
						this.H_indicators = [];
						this.C_indicators = [];
						alert('No data available for ' + name)
					} else {
						// fetch data for indicators table
						let link = 'https://covidtrackerapi.bsg.ox.ac.uk/api/v2/stringency/actions/' + code + '/' + this.date_function();
						axios.get(link).then(response => {
							this.H_indicators = [];
							this.C_indicators = [];
							for (let record in response.data.policyActions) {
								let polCode = response.data.policyActions[record].policy_type_code
								let describe = response.data.policyActions[record].policy_type_display
								let val = response.data.policyActions[record].policyvalue
								let pol = response.data.policyActions[record].policy_value_display_field
								const row = {
									code: polCode,
									descr: describe,
									value: val,
									policy: pol
								}
								// populate table
								if (polCode.slice(0, 1) === 'H') {
									this.H_indicators.push(row)
								} else if (polCode.slice(0, 1) === 'C') {
									this.C_indicators.push(row)
								}
							}
						})
					}
				})
				
			}
		},
		created() {
			this.updateData('[AFG] Afghanistan');
		},
		mounted() {
			this.createChart("radarChart", this.radarChartData);
			this.date = this.date_function()
		}
	}
</script>
