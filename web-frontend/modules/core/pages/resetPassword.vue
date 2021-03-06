<template>
  <div>
    <div v-if="!success">
      <h1 class="box__title">Reset password</h1>
      <Error :error="error"></Error>
      <form @submit.prevent="resetPassword">
        <div class="control">
          <label class="control__label">New password</label>
          <div class="control__elements">
            <input
              v-model="account.password"
              :class="{ 'input--error': $v.account.password.$error }"
              type="password"
              class="input input--large"
              @blur="$v.account.password.$touch()"
            />
            <div v-if="$v.account.password.$error" class="error">
              A new password is required.
            </div>
          </div>
        </div>
        <div class="control">
          <label class="control__label">Repeat new password</label>
          <div class="control__elements">
            <input
              v-model="account.passwordConfirm"
              :class="{ 'input--error': $v.account.passwordConfirm.$error }"
              type="password"
              class="input input--large"
              @blur="$v.account.passwordConfirm.$touch()"
            />
            <div v-if="$v.account.passwordConfirm.$error" class="error">
              This field must match your new password field.
            </div>
          </div>
        </div>
        <div class="actions">
          <ul class="action__links">
            <li>
              <nuxt-link :to="{ name: 'login' }">
                <i class="fas fa-arrow-left"></i>
                Back to login
              </nuxt-link>
            </li>
          </ul>
          <button
            :class="{ 'button--loading': loading }"
            class="button button--large"
            :disabled="loading"
          >
            Change password
            <i class="fas fa-pencil-alt"></i>
          </button>
        </div>
      </form>
    </div>
    <div v-if="success" class="box__message">
      <div class="box__message-icon">
        <i class="fas fa-check"></i>
      </div>
      <h1 class="box__message-title">Password changed</h1>
      <nuxt-link :to="{ name: 'login' }" class="button button--large">
        <i class="fas fa-arrow-left"></i>
        Back to login
      </nuxt-link>
    </div>
  </div>
</template>

<script>
import { required, sameAs } from 'vuelidate/lib/validators'

import { ResponseErrorMessage } from '@baserow/modules/core/plugins/clientHandler'
import error from '@baserow/modules/core/mixins/error'
import AuthService from '@baserow/modules/core/services/auth'

export default {
  layout: 'login',
  mixins: [error],
  data() {
    return {
      loading: false,
      success: false,
      account: {
        password: '',
        passwordConfirm: '',
      },
    }
  },
  methods: {
    async resetPassword() {
      this.$v.$touch()
      if (this.$v.$invalid) {
        return
      }

      this.loading = true
      this.hideError()

      try {
        const token = this.$route.params.token
        await AuthService(this.$client).resetPassword(
          token,
          this.account.password
        )
        this.success = true
        this.loading = false
      } catch (error) {
        this.loading = false
        this.handleError(error, 'resetPassword', {
          BAD_TOKEN_SIGNATURE: new ResponseErrorMessage(
            'Invalid link.',
            'Could not reset the password because the link is invalid.'
          ),
          EXPIRED_TOKEN_SIGNATURE: new ResponseErrorMessage(
            'Link expired',
            'The password reset link has expired. Please request another one.'
          ),
        })
      }
    },
  },
  head() {
    return {
      title: 'Reset password',
    }
  },
  validations: {
    account: {
      password: { required },
      passwordConfirm: {
        sameAsPassword: sameAs('password'),
      },
    },
  },
}
</script>
