<template>
    <div class = "modal">
        <div class = "modal-content">
            <div @click = "$emit('close')" class = "close">close</div>
            <h3>Reset Password</h3>
            <div v-if = "!showSuccess">
                <p>Enter your email to reset your password</p>
                <form @submit.prevent>
                    <label>
                        <input placeholder = "you@email.com" type = "email" v-model.trim = "email"/>
                    </label>
                </form>
                <p class = "error" v-if = "errorMsg !== ''">{{ errorMsg }}</p>
                <md-button @click = "resetPassword()" class = "md-raised md-accent"
                           style="padding: 8px; width: 100%; height: 100%; display: block; overflow: hidden; margin: 0">
                    Reset
                </md-button>
            </div>
            <p v-else>Success! Check your email for a reset link.</p>
        </div>
    </div>
</template>

<script>
	import {auth} from '@/firebase'
	
	export default {
		data() {
			return {
				email: '',
				showSuccess: false,
				errorMsg: ''
			}
		},
		methods: {
			async resetPassword() {
				this.errorMsg = ''
				try {
					await auth.sendPasswordResetEmail(this.email)
					this.showSuccess = true
				} catch (err) {
					this.errorMsg = err.message
				}
			}
		}
	}
</script>
