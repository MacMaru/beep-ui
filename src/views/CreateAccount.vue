<template>
  <v-card>
    <v-form @submit.prevent="createAccount" ref="form" v-model="valid">
      <v-card-title>Create account</v-card-title>
      <v-card-text>
        <v-alert
          v-for="error in errors"
          :key="error.name"
          type="error"
          outlined
        >
          {{ error.type }}
        </v-alert>
        <v-text-field
          label="email"
          v-model="credentials.username"
          autocomplete="off"
          :rules="emailRules"
        ></v-text-field>
        <v-text-field
          label="password"
          type="password"
          v-model="credentials.password"
          :rules="passwordRules"
        ></v-text-field>
        <v-text-field
          label="repeat password"
          type="password"
          v-model="repeatedPassword"
          :rules="repeatPasswordRules"
        ></v-text-field>
      </v-card-text>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn text type="submit" :disabled="!valid">Create account</v-btn>
      </v-card-actions>
    </v-form>
  </v-card>
</template>

<script>
export default {
  methods: {
    async createAccount() {
      if (this.$refs.form.validate()) {
        this.clearErrors()
        try {
          await this.$store.dispatch('Auth/signUp', this.credentials)
          await this.$router.push({
            name: 'confirmSignUp',
            query: { email: this.credentials.username },
          })
        } catch (error) {
          switch (error.code) {
            case 'UsernameExistsException':
              this.errors.push({
                type: 'error.user_already_exists',
              })
              break
            case 'LimitExceededException':
              this.errors.push({
                type: 'error.limit_exceeded_try_again_later',
              })
              break
            case 'ResourceNotFoundException':
              this.errors.push({
                type: 'error.user_pool_client_not_found',
              })
              break
            default:
              this.errors.push({
                type: 'error.unknown_error',
              })
          }
        }
      }
    },
    clearErrors() {
      this.errors = []
    },
  },
  computed: {
    hasErrors() {
      return this.errors.length > 0
    },
  },
  data() {
    return {
      credentials: {},
      valid: false,
      repeatedPassword: '',
      errors: [],
      emailRules: [
        v => !!v || 'error.email_required',
        v => /.+@.+\..+/.test(v) || 'error.invalid_email',
      ],
      verificationCodeRules: [v => !!v || 'error.verification_code_required'],
      passwordRules: [
        v => !!v || 'error.password_required',
        v =>
          /^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[\^$*.[\]{}()?\-"!@#%&/\\,><':;|_~`])(?=.{6,98})/.test(
            v
          ) || 'error.password_does_not_match_policy',
      ],
      repeatPasswordRules: [
        v => !!v || 'error.field_required',
        v => v === this.credentials.password || 'error.passwords_do_not_match',
      ],
    }
  },
}
</script>

<style scoped></style>
