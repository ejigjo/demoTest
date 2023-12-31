<script setup>
import { ref } from 'vue'
import { useUserInfoStore } from '@/stores/userInfo.js';
const userInfoStore = useUserInfoStore();
const userInfo = ref({ ...userInfoStore.info })

//定義重設密碼數據
const rePasswordData = ref({
    old_pwd: '', // 對應後端的 old_pwd
    new_pwd: '', // 對應後端的 new_pwd
    re_pwd: ''   // 對應後端的 re_pwd
})

//校驗密碼函數
const checkRePassword = (rule, value, callback) => {
    if (value === '') {
        callback(new Error('請再確認輸入密碼'))
    } else if (value != rePasswordData.value.newPassword) {
        callback(new Error('兩次密碼輸入不一致'))
    } else {
        callback()
    }
}
//定義重新設置密碼規則
const rules = {
    old_pwd: [
        { required: true, message: '請輸入原密碼', trigger: 'blur' },
        { min: 5, max: 16, message: '長度為5~16位非空字符', trigger: 'blur' }
    ],
    new_pwd: [
        { required: true, message: '請輸入新密碼', trigger: 'blur' },
        { min: 5, max: 16, message: '長度為5~16位非空字符', trigger: 'blur' }
    ],
    re_pwd: [
        { validator: checkRePassword, trigger: 'blur' }
    ]
}

//修改密碼
import { userUpdatePwdService } from '@/api/user.js'
import { ElMessage } from 'element-plus';
const updateUserPwd = async () => {
    let result = await userUpdatePwdService(rePasswordData.value)
    ElMessage.success(result.msg ? result.msg : '修改成功');
    //修改pinia密碼
    userInfoStore.setInfo(userInfo.value);

}


</script>
<template>
    <el-card class="page-container">
        <template #header>
            <div class="header">
                <span>重設密碼</span>
            </div>
        </template>
        <el-row>
            <el-col :span="12">
                <el-form :rules="rules" label-width="100px" size="large" :model="rePasswordData">
                    <el-form-item label="原密碼" prop="password">
                        <el-input v-model="rePasswordData.old_pwd"></el-input>
                    </el-form-item>
                    <el-form-item label="新密碼" prop="newPassword">
                        <el-input v-model="rePasswordData.new_pwd"></el-input>
                    </el-form-item>
                    <el-form-item label="確認新密碼" prop="rePassword">
                        <el-input v-model="rePasswordData.re_pwd"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="updateUserPwd">提交修改</el-button>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
    </el-card>
</template>