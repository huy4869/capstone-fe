<template>
  <div class="main-login">
    <div>
      <div class="login login-width login-mobile">
        <div class="d-flex justify-center mb-8">
          <img src="@/assets/images/login/login-icon.svg" alt="">
        </div>
        <h3 class="title text-center text-[#011A51] font-semibold">{{ $t('account.login_title') }}</h3>
        <el-form
          ref="accountForm"
          :model="accountForm"
          :rules="accountRules"
          autocomplete="off"
          label-position="left"
          @keyup.enter.native="login"
        >
          <el-form-item class="email-login" prop="phone" :error="(error.key === 'phone') ? error.value : ''">
            <label for="email">{{ $t('account.phone') }}</label>
            <div class="custom-register-input">
              <el-input
                id="phone"
                ref="phone"
                v-model.trim="accountForm.phone"
                :placeholder="$t('account.phone')"
                autocomplete="off"
                name="phone"
                type="text"
                tabindex="2"
                maxlength="12"
                oninput="this.value=this.value.replace(/[^0-9]/g,'');"
                pattern="[0-9]*"
                inputmode="numeric"
                @focus="resetValidate('phone')"
              >
                <template #prefix>
                  <div class="d-flex items-center px-[10px]" style="height: 100%">
                    <img src="@/assets/images/register/vietnam.svg" alt="" class="rounded-sm" style="width: 21px; height: 14px;"/>
                    <span class="pl-[8px] text-[#606266]">+84</span>
                  </div>
                </template>
              </el-input>
            </div>
          </el-form-item>
          <el-form-item class="email-login" prop="password" :error="(error.key === 'password') ? error.value : ''">
            <label for="password">{{ $t('account.password') }}</label>
            <el-input
              id="password"
              ref="password"
              v-model="accountForm.password"
              :placeholder="$t('account.password')"
              name="password"
              :type="showPass?'text':'password'"
              tabindex="3"
              maxlength="32"
              autocomplete="off"
              @focus="resetValidate('password')"
            >
              <i slot="suffix" class="cursor-pointer" @click="displayPass()">
                <img v-if="showPass" class="icon-show-pass" src="~/assets/images/icons/eye-input.svg" alt="showpass"/>
                <img v-else class="icon-show-pass" src="@/assets/images/icons/hide-eye.svg" alt="hidepass"/>
              </i>
            </el-input>
          </el-form-item>

<!--          <el-form-item :error="(error.key === 'g-recaptcha-response') ? error.value : ''" prop="captcha" class="captcha">-->
<!--            <template>-->
<!--              <recaptcha-->
<!--                ref="captcha"-->
<!--                @error="onError"-->
<!--                @success="onSuccess"-->
<!--                @expired="onExpired"-->
<!--              />-->
<!--            </template>-->
<!--          </el-form-item>-->
          <div class="d-flex align-items-center forgot-pass">
            <div
              class="content cursor-pointer login-page__forgot-password align-items-center text-sm" @click="$router.push('/forgot-pass')">
              {{ $t('account.forgot_password') }}
            </div>
          </div>
          <el-form-item>
            <div :class="{'disabled' : disabledButton, 'common-button': 'common-button'}">
              <el-button
                v-loading.fullscreen.lock="fullscreenLoading"
                :loading="loading"
                type="custom-primary"
                :disabled="disabledButton"
                @click.native="login"
              >
                {{ $t('account.login') }}
              </el-button>
            </div>
          </el-form-item>
          <div class="d-flex align-items-center no_account">
            <div
              class="content text-center cursor-pointer login-page__forgot-password align-items-center" @click="step=3">
              {{ $t('account.no_account') }} <span class="cursor-pointer underline text-[#344874]" @click="handeRegister">{{ $t('account.here')  }}</span>
            </div>
          </div>
        </el-form>
      </div>
    </div>
  </div>
</template>
<script>
import _ from 'lodash'
import { INDEX_SET_ERROR, INDEX_SET_LOADING } from '@/store/store.const'
import { validPhoneNoPrefix } from '@/utils/validate'

export default {
  name: 'LoginPage',
  layout: 'none',
  data() {
    const validPhoneNumber = (rule, value, callback) => {
      if (value == null || value === '') {
        callback()
      } else if (!validPhoneNoPrefix(value)) {
        callback(new Error(this.$t('validation.phone_length')))
      } else {
        callback()
      }
    }

    const validatePass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error(this.$t('validation.required', { _field_: this.$t('account.password') }).toString()))
      } else {
        if (value.length > 20 || value.length < 8) {
          callback(new Error(this.$t('validation.pass_format')))
        }
        if (this.accountForm.password_confirmation !== '') {
          this.$refs.accountForm.validateField('password_confirmation')
        }
        callback()
      }
    }
    return {
      token: '',
      user: {},
      accountForm: {
        phone: '',
        password: '',
        remember: '',
        errors: {}
      },
      error: {
        key: null,
        value: ''
      },
      accountRules: {
        phone: [
          {
            required: true,
            message: this.$t('validation.required', { _field_: this.$t('account.phone') }),
            trigger: 'blur'
          },
          {
            validator: validPhoneNumber, trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: this.$t('validation.required', { _field_: this.$t('account.password') }),
            trigger: 'blur'
          },
          { validator: validatePass, trigger: 'blur' }
        ],
        remember: []
      },
      valid: false,
      loading: false,
      fullscreenLoading: false,
      showPass: false,
      isValid: false
    }
  },
  async created() {
    await this.checkLogin()
  },
  computed: {
    disabledButton() {
      return this.accountForm.phone === '' || this.accountForm.password === ''
    }
  },
  methods: {
    checkLogin() {
      if (this.$auth.loggedIn) {
        this.$router.push('/event')
      }
    },
    handeRegister() {
      this.$router.push('/register')
    },
    resetValidate(ref) {
      if (ref === this.error.key) {
        this.error = { key: null, value: '' }
      }
      this.$refs.accountForm.fields.find((f) => f.prop === ref).clearValidate()
      this.accountForm.errors[ref] = ''
    },
    checkCapslock(e, attr) {
      const { key } = e
      this[attr] = key && key.length === 1 && (key >= 'A' && key <= 'Z')
    },
    async login() {
      this.error = { key: null, value: '' }
      this.validateForm()
      if (!this.isValid) {
        return
      }

      try {
        await this.$store.commit(INDEX_SET_LOADING, true)
        const dto = _.cloneDeep(this.accountForm)
        const result = await this.$auth.loginWith('local', {
          data: {
            phoneNumber: '+84' + (dto.phone.startsWith('0') ? dto.phone.slice(1, dto.phone.length) : dto.phone),
            password: this.accountForm.password
          }
        })

        const data = result.data
        switch (data.statusCode) {
          case 202:
            await this.$router.push('/event')
            break
          default:
            this.$store.commit(INDEX_SET_ERROR, { show: true, text: 'L???i !', message: data.message })
            break
        }
      } catch (err) {
        this.$store.commit(INDEX_SET_ERROR, { show: true, text: 'L???i !', message: this.$t('message.message_error') })
      }
      await this.$store.commit(INDEX_SET_LOADING, false)
    },
    validateForm() {
      this.$refs.accountForm.validate(valid => {
        this.isValid = valid
      })
    },
    displayPass() {
      this.showPass = !this.showPass
    },
    closeModalErr() {
      this.isAuthErr = false
    }
  }
}
</script>
<style>
/* stylelint-disable */
.custom-register-input .el-input--prefix .el-input__inner {
  padding-left: 80px;
}
/* stylelint-enable */
</style>
