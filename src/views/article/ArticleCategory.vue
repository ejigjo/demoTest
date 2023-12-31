<script setup>
import {
     Edit,
     Delete
} from '@element-plus/icons-vue'
import { ref } from 'vue'
const categorys = ref([])
import { articleCategoryListService, articleCategoryAddService, articleCategoryUpdateService, articleCategoryDeleteService } from '@/api/article.js'
const articleCategoryList = async () => {
     let result = await articleCategoryListService();
     categorys.value = result.data;
}
articleCategoryList();
//控制新增分類彈窗
const dialogVisible = ref(false)

//新增分類資料模型
const categoryModel = ref({
     categoryName: '',
     categoryAlias: ''
})
//新增分類表單校驗
const rules = {
     categoryName: [
         { required: true, message: '請輸入分類名稱', trigger: 'blur' },
     ],
     categoryAlias: [
         { required: true, message: '請輸入分類別名稱', trigger: 'blur' },
     ]
}


//呼叫API,新增表單
import { ElMessage } from 'element-plus'
const addCategory = async () => {

     let result = await articleCategoryAddService(categoryModel.value);
     ElMessage.success(result.msg ? result.msg : '新增成功')
     //重新載入列表
     articleCategoryList();
     //關閉新增視窗
     dialogVisible.value = false;
}
//定義彈跳視窗標題
const title = ref('')
//展示編輯彈跳視窗
const showDialog = (row) => {
     dialogVisible.value = true; title.value = '編輯分類'
     // 數據拷貝
     categoryModel.value.categoryName = row.categoryName;
     categoryModel.value.categoryAlias = row.categoryAlias;
     //增加id屬性,後台編輯資料需要
     categoryModel.value.id = row.id
}
//編輯分類
const updateCategory = async () => {

     let result = await articleCategoryUpdateService(categoryModel.value);
     ElMessage.success(result.msg ? result.msg : '編輯成功')
     //重新載入列表
     articleCategoryList();
     //關閉編輯視窗
     dialogVisible.value = false;
}
//清空表單
const clearData = () => {
     categoryModel.value.categoryName = '';
     categoryModel.value.categoryAlias = '';
}
import { ElMessageBox } from 'element-plus'
//刪除分類
const deleteCategory = (row) => {
     ElMessageBox.confirm(
         '你確認要刪除該分類嗎?',
         '提示',
         {
             confirmButtonText: '確認',
             cancelButtonText: '取消',
             type: 'warning',
         }
     )
         .then(async () => {
             let result = await articleCategoryDeleteService(row.id);
             ElMessage({
                 type: 'success',
                 message: '刪除成功',
             })
             //重新載入列表
             articleCategoryList();

         })
         .catch(() => {
             ElMessage({
                 type: 'info',
                 message: '刪除取消',
             })
         })
}
</script>


<template>
     <el-card class="page-container">
         <template #header>
             <div class="header">
                 <span>文章分類</span>
                 <div class="extra">
                     <el-button type="primary" @click="dialogVisible = true; title = '新增分類'; clearData()">新增分類</el-button>
                 </div>
             </div>
         </template>
         <el-table :data="categorys" style="width: 100%">
             <el-table-column label="序號" width="100" type="index"> </el-table-column>
             <el-table-column label="分類名稱" prop="categoryName"></el-table-column>
             <el-table-column label="分類別名" prop="categoryAlias"></el-table-column>
             <el-table-column label="操作" width="100">
                 <!-- 這裡的row代表一整列的數據 -->
                 <template #default="{ row }">
                     <!-- 修改刪除按鈕 -->
                     <el-button :icon="Edit" circle plain type="primary" @click="showDialog(row)"></el-button>
                     <el-button :icon="Delete" circle plain type="danger" @click="deleteCategory(row)"></el-button>
                 </template>
             </el-table-column>
             <template #empty>
                 <el-empty description="沒有數據" />
             </template>

         </el-table>

         <!-- 新增/編輯對話框 -->
         <el-dialog v-model="dialogVisible" :title="title" width="30%">
             <el-form :model="categoryModel" :rules="rules" label-width="100px" style="padding-right: 30px">
                 <el-form-item label="分類名稱" prop="categoryName">
                     <el-input v-model="categoryModel.categoryName" minlength="1" maxlength="10"></el-input>
                 </el-form-item>
                 <el-form-item label="分類別名" prop="categoryAlias">
                     <el-input v-model="categoryModel.categoryAlias" minlength="1" maxlength="15"></el-input>
                 </el-form-item>
             </el-form>
             <template #footer>
                 <span class="dialog-footer">
                     <el-button @click="dialogVisible = false">取消</el-button>
                     <el-button type="primary" @click="title == '新增分類' ? addCategory() : updateCategory()"> 確認 </el-button>
                 </span>
             </template>
         </el-dialog>
     </el-card>
</template>

<style lang="scss" scoped>
.page-container {
     min-height: 100%;
     box-sizing: border-box;

     .header {
         display: flex;
         align-items: center;
         justify-content: space-between;
     }
}
</style>