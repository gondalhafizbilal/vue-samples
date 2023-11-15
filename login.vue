<template>
  <div class="login">
    <div class="team-name hidden-sm-and-down">
      <img src="@/assets/image/login/team-name.png" alt="logo" />
    </div>
    <div
      class="form-box"
      v-loading="isLoading"
      element-loading-background="rgba(0, 0, 0, 0)"
    >
      <div class="title"><h1 title="Aurora">Aurora</h1></div>
      <form
        class="login-form"
        autocomplete="off"
        @submit.prevent="throttleLogin()"
      >
        <div class="form-item nickname">
          <span class="icon account-icon"></span>
          <input
            type="text"
            v-model="loginData.username"
            autocomplete="off"
            placeholder="Enter username"
          />
        </div>
        <div class="form-item password">
          <span class="icon secret-icon"></span>
          <input
            type="password"
            v-model="loginData.password"
            autocomplete="off"
            placeholder="Enter user password"
          />
        </div>
        <div class="form-item password" v-if="captchaImage">
          <img class="captcha" :src="captchaImage" @click.stop="getCaptcha()" />
          <input
            type="text"
            v-model="loginData.captcha"
            autocomplete="off"
            placeholder="Enter captcha"
          />
        </div>
        <button class="submit-btn" type="submit">Login</button>
      </form>
    </div>
  </div>
</template>

<script>
import { reactive, ref, onMounted } from "vue";
import { useStore } from "vuex";
import { useRouter } from "vue-router";
import { ElMessage } from "element-plus";
import axios from "aurora/plugin/axios";
import UserModel from "@/aurora/model/user";
import Utils from "@/aurora/util/util";
import Config from "@/config";

export default {
  setup() {
    let tag = "";
    const wait = 2000;
    const isLoading = ref(false);
    const captchaImage = ref("");
    const store = useStore();
    const router = useRouter();
    const throttleLogin = ref(null);

    const loginData = reactive({
      username: "",
      password: "",
      captcha: "",
    });

    const login = async () => {
      const { username, password, captcha } = loginData;
      try {
        isLoading.value = true;
        await UserModel.getToken(username, password, captcha, tag);
        await getInformation();
        isLoading.value = false;
        router.push(Config.defaultRoute);
        ElMessage({
          message: "Login successful",
          type: "success",
        });
      } catch (e) {
        getCaptcha();
        isLoading.value = false;
      }
    };

    const getCaptcha = async () => {
      axios({
        method: "POST",
        url: "cms/user/captcha",
      }).then((result) => {
        ({ tag } = result);
        captchaImage.value = result.image;
      });
    };

    const getInformation = async () => {
      try {
        const user = await UserModel.getPermissions();
        store.dispatch("setUserAndState", user);
        store.commit("SET_USER_PERMISSIONS", user.permissions);
      } catch (e) {
        console.error(e);
      }
    };

    onMounted(() => {
      getCaptcha();
      throttleLogin.value = Utils.throttle(login, wait);
    });

    return {
      loginData,
      isLoading,
      getCaptcha,
      captchaImage,
      throttleLogin,
    };
  },
};
</script>

<style lang="scss">
.login {
  width: 100%;
  height: 100%;
  background: #1b2c5f url("../../assets/image/login/login-ba.png") no-repeat
    center center;
  background-size: cover;

  .team-name {
    position: fixed;
    left: 40px;
    top: 50%;
    width: 50px;
    transform: translateY(-50%);
  }

  .form-box {
    position: fixed;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 445px;

    .title {
      height: 37px;
      font-size: 30px;
      line-height: 37px;
      margin-bottom: 15%;

      h1 {
        padding-left: 74px;
        box-sizing: border-box;
        text-align: left;
        color: #8c98ae;
      }
    }

    .login-form {
      width: 100%;

      .form-item {
        position: relative;
        width: 100%;
        height: 40px;
        box-sizing: border-box;
        padding-bottom: 13px;
        margin-bottom: 34px;

        input {
          width: 100%;
          height: 100%;
          background: transparent;
          color: #c4c9d2;
          font-size: 14px;
          padding-left: 74px;
          box-sizing: border-box;
        }

        .captcha {
          position: absolute;
          width: 80px;
          right: 30px;
          top: -22px;
          cursor: pointer;
        }
      }

      .form-item.nickname {
        background: url("../../assets/image/login/nickname.png") no-repeat;
        background-size: 100% auto;
        background-position: left bottom;
      }

      .form-item.password {
        background: url("../../assets/image/login/password.png") no-repeat;
        background-size: 100% auto;
        background-position: left bottom;
      }

      .submit-btn {
        width: 100%;
        height: 70px;
        color: #c4c9d2;
        font-size: 16px;
        text-align: left;
        box-sizing: border-box;
        padding: 0 10px;
        padding-left: 74px;
        background: url("../../assets/image/login/login-btn.png") no-repeat;
        background-size: 90% auto;
        background-position: center bottom;
        border: none;
        cursor: pointer;
      }
    }
  }
}
</style>