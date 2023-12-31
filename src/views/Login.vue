<script setup>
import { User, Lock } from '@element-plus/icons-vue'
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
//控制註冊與登入表單的顯示， 預設顯示登錄
const isRegister = ref(false)
//定義註冊數據
const registerData = ref({
    username: [],
    password: [],
    rePassword: []
})
//校驗密碼函數
const checkRePassword = (rule, value, callback) => {
    if (value === '') {
        callback(new Error('請再確認輸入密碼'))
    } else if (value != registerData.value.password) {
        callback(new Error('兩次密碼輸入不一致'))
    } else {
        callback()
    }
}
//定義註冊表單輸入規則
const rules = {
    username: [
        { required: true, message: '請輸入用戶名', trigger: 'blur' },
        { min: 5, max: 16, message: '長度為5~16位非空字符', trigger: 'blur' }
    ],
    password: [
        { required: true, message: '請輸入密碼', trigger: 'blur' },
        { min: 5, max: 16, message: '長度為5~16位非空字符', trigger: 'blur' }
    ],
    rePassword: [
        { validator: checkRePassword, trigger: 'blur' }
    ]

}


//調用後台API,完成註冊
import { userRegisterService, userLoginService } from '@/api/user';
const register = async () => {

    let result = await userRegisterService(registerData.value);
    ElMessage.success(result.msg ? result.msg : '註冊成功');
    clearRegisterData()
    isRegister.value = false;
}
import { useTokenStore } from '@/stores/token.js'
import { useRouter } from 'vue-router'
const router = useRouter();
const tokenStore = useTokenStore();
// 調用後台API,完成登錄
const login = async () => {

    let result = await userLoginService(registerData.value);
    ElMessage.success(result.msg ? result.msg : '登錄成功');
    tokenStore.setToken(result.data);
    router.push('/')
}

const clearRegisterData = () => {
    registerData.value = {
        username: '',
        password: '',
        rePassword: ''
    }
}


</script>

<template>
    <el-row class="login-page">
        <el-col :span="0" class="form-container"></el-col>
        <el-col :span="6" class="form">
            <!-- 註冊表單 -->
            <el-form ref="form" size="large" autocomplete="off" v-if="isRegister" :model="registerData" :rules="rules">
                <el-form-item>
                    <h1>註冊</h1>
                </el-form-item>
                <el-form-item prop="username">
                    <el-input :prefix-icon="User" placeholder="請輸入用戶名" v-model="registerData.username"></el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input :prefix-icon="Lock" type="password" placeholder="請輸入密碼"
                        v-model="registerData.password"></el-input>
                </el-form-item>
                <el-form-item prop="rePassword">
                    <el-input :prefix-icon="Lock" type="password" placeholder="請輸入再次密碼"
                        v-model="registerData.rePassword"></el-input>
                </el-form-item>
                <!-- 註冊按鈕 -->
                <el-form-item>
                    <el-button class="button" type="primary" auto-insert-space @click="register">
                        註冊
                    </el-button>
                </el-form-item>
                <el-form-item class="flex">
                    <el-link type="info" :underline="false" @click="isRegister = false; clearRegisterData()">
                        ← 返回
                    </el-link>
                </el-form-item>
            </el-form>
            <!-- 登入表單 -->
            <el-form ref="form" size="large" autocomplete="off" v-else :model="registerData" :rules="rules">
                <el-form-item>
                    <h1>登入</h1>
                </el-form-item>
                <el-form-item prop="username">
                    <el-input :prefix-icon="User" placeholder="請輸入使用者名稱" v-model="registerData.username"></el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input name="password" :prefix-icon="Lock" type="password" placeholder="請輸入密碼"
                        v-model="registerData.password"></el-input>
                </el-form-item>
                <el-form-item class="flex">
                    <div class="flex">
                
                        <el-link type="primary" :underline="false">忘記密碼？ </el-link>
                    </div>
                </el-form-item>
                <!-- 登入按鈕 -->
                <el-form-item>
                    <el-button class="button" type="primary" auto-insert-space @click="login">登入</el-button>
                </el-form-item>
                <el-form-item class="flex">
                    <el-link type="info" :underline="false" @click="isRegister = true; clearRegisterData()">
                        註冊 →
                    </el-link>
                </el-form-item>
            </el-form>
        </el-col>
    </el-row>
</template>

<style lang="scss" scoped>
/* 樣式 */
.login-page {
    height: 100vh;
    background: url('@/assets/login_bg.jpg') no-repeat center / cover;
    /* 背景圖片 */
    display: flex;
    align-items: center;
    /* 將內容垂直置中 */
    justify-content: center;
    /* 將內容水平置中 */
}

.form-container {
    justify-content: center;
    /* 將內容水平置中 */
    align-items: center;
    /* 將內容垂直置中 */
    height: 100vh;
    width: 100%;
    /* 使 .form-container 寬度充滿其父容器 */
}

.form {
    display: flex;
    flex-direction: column;
    justify-content: center;
    user-select: none;
    /* 禁用用戶選擇 */
   background-color: hsla(0, 0%, 0%, 0.5);

    padding: 20px;
    border-radius: 10px;
    width: 100%;
    /* 調整表單的寬度，可以根據需要進行調整 */
    max-width: 400px;
    /* 設定最大寬度 */

    .title {
        margin: 0 auto;
        /* 將標題置中 */
        text-align: right; /* 將文字向右對齊 */
        margin-right: 50px; /* 向右移動的間距 */
    }

    .button {
        width: 100%;
        /* 將按鈕寬度設為100% */
    }

    .flex {
        width: 100%;
        display: flex;
        justify-content: space-between;
        /* 將內容平均分配空間 */
    }

    el-form-item {
        margin-bottom: 20px;
        /* 在表單項目之間添加一些間距 */
    }

    h1 {
        text-align: center;
        color: rgb(0, 0, 0);
        font-weight: bold;
    }
    .flex el-checkbox {
        color: white; /* 修改文字顏色 */
    }
    .background-image {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    overflow: hidden;
    z-index: -1; /* 將其置於背景之下 */
}

.background-image img {
    width: 100%;
    height: 100%;
    object-fit: cover; /* 保持圖片比例並填充 */
}
   
}</style>
