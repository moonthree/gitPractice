<template>
  <el-dialog custom-class="login-dialog" title="로그인" v-model="state.dialogVisible" @close="handleClose">
    <el-form :model="state.form" :rules="state.rules" ref="loginForm" :label-position="state.form.align">
      <el-form-item prop="id" label="아이디" :label-width="state.formLabelWidth" >
        <el-input v-model="state.form.id" @input="onChange()" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item prop="password" label="비밀번호" :label-width="state.formLabelWidth">
        <el-input v-model="state.form.password"  @input="onChange()" autocomplete="off" show-password></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button type="primary" @click="clickLogin">로그인</el-button>
      </span>
    </template>
  </el-dialog>
</template>
<style>
.login-dialog {
  width: 400px !important;
  height: 300px;
}
.login-dialog .el-dialog__headerbtn {
  float: right;
}
.login-dialog .el-form-item__content {
  margin-left: 0 !important;
  float: right;
  width: 200px;
  display: inline-block;
}
.login-dialog .el-form-item {
  margin-bottom: 20px;
}
.login-dialog .el-form-item__error {
  font-size: 12px;
  color: red;
}
.login-dialog .el-input__suffix {
  display: none;
}
.login-dialog .el-dialog__footer {
  margin: 0 calc(50% - 80px);
  padding-top: 0;
  display: inline-block;
}
.login-dialog .dialog-footer .el-button {
  width: 120px;
}
</style>
<script>
import { reactive, computed, ref, onMounted } from 'vue'
import { useStore } from 'vuex'
import { ElLoading } from 'element-plus'


export default {
  name: 'login-dialog',

  props: {
    open: {
      type: Boolean,
      default: false
    }
  },

  setup(props, { emit }) {
    const store = useStore()
    // 마운드 이후 바인딩 될 예정 - 컨텍스트에 노출시켜야함. <return>
    const loginForm = ref(null)

    /*
      // Element UI Validator
      // rules의 객체 키 값과 form의 객체 키 값이 같아야 매칭되어 적용됨
      //
    */
    const state = reactive({
      form: {
        id: '',
        password: '',
        align: 'left'
      },
      rules: {
        id: [
          { required: true, message: 'Please input ID', trigger: 'blur' },
          { max: 16, message: '최대 16자까지 입력 가능합니다.', trigger: 'blur'}
        ],
        password: [
          { trigger: 'blur', validator (rule, value, callback) {
            if (/^.*(?=^)(?=.*\d)(?=.*[a-zA-Z])(?=.*[!@#$%^&+=]).*$/.test(value)) {
              callback();
            } else {
              callback(new Error('비밀번호는 영문, 숫자, 특수문자가 조합되어야 합니다.'))
            }
          }},
          { required: true, message: 'Please input password', trigger: 'blur' },
          { max: 16, message: '최대 16자까지 입력 가능합니다.', trigger: 'blur'},
          { min: 9, message: '최소 9자를 입력 해야합니다.', trigger: 'blur'}
        ],
      },
      dialogVisible: computed(() => props.open),
      formLabelWidth: '120px'
    })

    onMounted(() => {
      // console.log(loginForm.value)
    })
    const onChange = function() {
      loginForm.value.validate(async () => {
        await console.log()
      })
    }

    const clickLogin = function () {
      // 로그인 클릭 시 validate 체크 후 그 결과 값에 따라, 로그인 API 호출 또는 경고창 표시
      loginForm.value.validate(async (valid) => {
        const loadingInstance = ElLoading.service({
          lock: true,
          text: 'Loading',
          background: 'rgba(0, 0, 0, 0.7)',
        })
        if (valid) {
          console.log('submit')
          await store.dispatch('accountStore/loginAction', { id: state.form.id, password: state.form.password })
          console.log('local')
          await localStorage.setItem('token',store.getters['accountStore/getToken'] )
          console.log('getme')
          await store.dispatch('accountStore/getMeAction', store.getters['accountStore/getToken'])
          await emit('closeLoginDialog')
          // console.log('accessToken ' + store.getters['accountStore/getToken'])
          loadingInstance.close()
        } else {
          alert('Validate error!')
        }
      });
    }

    const handleClose = function () {
      state.form.id = ''
      state.form.password = ''
      emit('closeLoginDialog')
    }

    return { loginForm, state, clickLogin, handleClose, onChange }
  }
}
</script>