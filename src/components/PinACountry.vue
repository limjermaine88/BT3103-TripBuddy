<template>
    <div id = "statistics_form">
        <section>
            <md-button @click = "pinACountry(); $bvToast.show('my-toast')" class = "md-raised md-accent"
                       style="padding: 8px; width: 100%; height: 100%; display: block; overflow: hidden; margin: 0">
                Pin a country!
            </md-button>
            <b-toast @click = "validPinnedCountry = false" id = "my-toast" solid variant = "warning">
                <template #toast-title>
                    <div class = "d-flex flex-grow-1 align-items-baseline">
                        <b-img blank blank-color = "#ff5555" class = "mr-2" height = "14" width = "14"></b-img>
                        <strong class = "mr-auto">{{status}}!</strong>
                        <small class = "text-muted mr-2">1 seconds ago</small>
                    </div>
                </template>
                <p>
                    {{textMessage}}<br>
                    To edit full pinned country list, please proceed to
                    <router-link to = "/settings">Settings page</router-link>
                </p>
            </b-toast>
        </section>
    </div>
</template>

<script>
	import {mapState} from "vuex";
	
	export default {
		data() {
			return {
				validPinnedCountry: false,
				textMessage: '',
				status: ''
			}
		},
		props: {
			selectedCountry: {
				type: String
			}
		},
		methods: {
			pinACountry() {
				console.log("validPinnedCountry: ", this.validPinnedCountry)
				let currentInterestedCountry = this.$store.state.userProfile.country_interested;
				console.log("currentInterestedCountry: ", currentInterestedCountry)
				if (!currentInterestedCountry.includes(this.selectedCountry)
					&& this.country_options_dropdown.includes(this.selectedCountry)) {
					currentInterestedCountry.push(this.selectedCountry)
					this.$store.dispatch('updateProfile', {
						email: this.userProfile.email,
						name: this.userProfile.name,
						nationality: this.userProfile.nationality,
						country_interested: currentInterestedCountry
					})
					this.status = "SUCCESS"
					this.textMessage = this.selectedCountry + " has been added to your customized pinned list of countries."
				} else if (this.selectedCountry !== '') {
					this.status = "FAIL"
					this.textMessage = this.selectedCountry + " has NOT been added to your customized pinned " +
						"list of countries because this target country is already in the pinned list OR its name is not in pre-defined list."
				} else {
					this.status = "FAIL"
					this.textMessage = "The target country is not defined"
				}
			}
		},
		computed: {
			...mapState(['userProfile', 'country_options_dropdown']),
		}
	}
</script>