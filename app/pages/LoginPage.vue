<template>
	<Page @loaded="checkLoginStatus()">
    <ActionBar>
      <barContent />
    </ActionBar>
		<FlexboxLayout class="page">
			<StackLayout class="form">
			  <Image class="logo" src="~/images/logo.png" />
				<!--<Label class="header" text="DEEP ENGLISH" />-->

				<StackLayout class="input-field" marginBottom="25">
					<TextField class="input" hint="Email" keyboardType="email" autocorrect="false" autocapitalizationType="none" v-model="user.email"
					 returnKeyType="next" @returnPress="focusPassword" fontSize="18" />
					<StackLayout class="hr-light" />
				</StackLayout>

				<StackLayout class="input-field" marginBottom="25">
					<TextField ref="password" class="input" hint="Password" secure="true" v-model="user.password" :returnKeyType="isLoggingIn ? 'done' : 'next'"
					 @returnPress="focusConfirmPassword" fontSize="18" />
					<StackLayout class="hr-light" />
				</StackLayout>

				<StackLayout v-show="!isLoggingIn" class="input-field">
					<TextField ref="confirmPassword" class="input" hint="Confirm password" secure="true" v-model="user.confirmPassword" returnKeyType="done"
					 fontSize="18" />
					<StackLayout class="hr-light" />
				</StackLayout>

				<Button :text="isLoggingIn ? 'Log In' : 'Sign Up'" @tap="submit" class="btn btn-warning m-t-20" />

				<!--<Label v-show="isLoggingIn" text="Forgot your password?" class="login-label" @tap="forgotPassword" />-->
			</StackLayout>

			<!--<Label class="login-label sign-up-label" @tap="toggleForm">
	          <FormattedString>
	            <Span :text="isLoggingIn ? 'Don’t have an account? ' : 'Back to Login'" />
	            <Span :text="isLoggingIn ? 'Sign up' : ''" class="bold" />
	          </FormattedString>
	        </Label>-->
		</FlexboxLayout>
	</Page>
</template>

<script>
import barContent from '~/components/barContent';
// A stub for a service that authenticates users.
const userService = {
    register(user) {
        return Promise.resolve(user);
    },
    login(user) {
        return Promise.resolve(user);
    },
    resetPassword(email) {
        return Promise.resolve(email);
    }
};

// A stub for the main page of your app. In a real app you’d put this page in its own .vue file.
const HomePage = {
    template: `
<Page>
	<Label class="m-20" textWrap="true" text="You have successfully authenticated. This is where you build your core application functionality."></Label>
</Page>
`
};

export default {
  components: {
    barContent
  },
    data() {
        return {
            isLoggingIn: true,
            user: {
                email: "",
                password: "",
                confirmPassword: ""
            }
        };
    },
		computed: {
			authuser () { return (this.$store.state.auth || {}).user || null },
		},
    methods: {
        toggleForm() {
            this.isLoggingIn = !this.isLoggingIn;
        },

        submit() {
            if (!this.user.email || !this.user.password) {
                this.alert(
                    "Please provide both an email address and password."
                );
                return;
            }
            if (this.isLoggingIn) {
							const payload = {
								email: this.user.email, // this.email, msahajwani@gmail.com
								password: this.user.password // this.password test123
							}
							/*const loginuser = await this.$store.dispatch('loginUser', payload)
							if (loginuser.data.success === true) {
								this.loading = false
								this.$router.push('/')
							}*/
							let login = this.$store.dispatch("loginUser", payload)
							console.log('post login response: ', login)
							if (login) {
								this.$navigateTo(this.$routes.Home);
							}
								//this.login();
            } else {
                this.register();
            }
        },
				checkLoginStatus () {
					if (this.authuser) {
						this.$navigateTo(this.$routes.My);
					}
				},
        login() {
            userService
                .login(this.user)
                .then(() => {
                    this.$navigateTo(this.$routes.My);
                })
                .catch(() => {
                    this.alert("Unfortunately we could not find your account.");
                });
        },
        register() {
            if (this.user.password != this.user.confirmPassword) {
                this.alert("Your passwords do not match.");
                return;
            }

            userService
                .register(this.user)
                .then(() => {
                    this.alert("Your account was successfully created.");
                    this.isLoggingIn = true;
                })
                .catch(() => {
                    this.alert(
                        "Unfortunately we were unable to create your account."
                    );
                });
        },

        forgotPassword() {
            prompt({
                title: "Forgot Password",
                message:
                    "Enter the email address you used to register for APP NAME to reset your password.",
                inputType: "email",
                defaultText: "",
                okButtonText: "Ok",
                cancelButtonText: "Cancel"
            }).then(data => {
                if (data.result) {
                    userService
                        .resetPassword(data.text.trim())
                        .then(() => {
                            this.alert(
                                "Your password was successfully reset. Please check your email for instructions on choosing a new password."
                            );
                        })
                        .catch(() => {
                            this.alert(
                                "Unfortunately, an error occurred resetting your password."
                            );
                        });
                }
            });
        },

        focusPassword() {
            this.$refs.password.nativeView.focus();
        },
        focusConfirmPassword() {
            if (!this.isLoggingIn) {
                this.$refs.confirmPassword.nativeView.focus();
            }
        },

        alert(message) {
            return alert({
                title: "APP NAME",
                okButtonText: "OK",
                message: message
            });
        }
    }
};
</script>

<style scoped>
	.page {
		align-items: center;
		flex-direction: column;
	}

	.form {
		margin-left: 30;
		margin-right: 30;
		flex-grow: 2;
		vertical-align: middle;
	}

	.logo {
		margin-bottom: 30;
		height: 70;
		font-weight: bold;
	}

	.header {
		horizontal-align: center;
		font-size: 25;
		font-weight: 600;
		margin-bottom: 70;
		text-align: center;
		color: #D51A1A;
	}

	.input-field {
		margin-bottom: 25;
	}

	.input {
		font-size: 18;
		placeholder-color: #A8A8A8;
	}

	.input-field .input {
		font-size: 54;
	}

	.btn-warning {
		height: 50;
		margin: 30 5 15 5;
		background-color: #f1c40f;
    color:#1e1e1e;
		border-radius: 5;
		font-size: 20;
		font-weight: 600;
	}

	.login-label {
		horizontal-align: center;
		color: #A8A8A8;
		font-size: 16;
	}

	.sign-up-label {
		margin-bottom: 20;
	}

	.bold {
		color: #000000;
	}
</style>
