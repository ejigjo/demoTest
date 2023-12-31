<script setup>
import {
    Edit,
    Delete
} from '@element-plus/icons-vue'

import { ref } from 'vue'

//文章分類資料模型
const categorys = ref([])

//使用者搜尋時選取的分類id
const categoryId = ref('')

//使用者搜尋時選取的發佈狀態
const state = ref('')

//文章列表資料模型
const articles = ref([])

//分頁條資料模型
const pageNum = ref(1)//目前頁
const total = ref(20)//總條數
const pageSize = ref(5)//每頁條數

//當每頁條數發生了變化，呼叫此函數
const onSizeChange = (size) => {
    pageSize.value = size
    articleList();
}
//當前頁碼發生變化，呼叫此函數
const onCurrentChange = (num) => {
    pageNum.value = num
    articleList();
}
//回顯文章分類
import { articleCategoryListService, articleListService, articleAddService, articleDeleteService, articleUpdateService } from '@/api/article.js'
const articleCategoryList = async () => {
    let result = await articleCategoryListService();

    categorys.value = result.data;
}

//取得文章列數據
const articleList = async () => {
    let params = {
        pageNum: pageNum.value,
        pageSize: pageSize.value,
        categoryId: categoryId.value ? categoryId.value : null,
        state: state.value ? state.value : null
    }
    let result = await articleListService(params);
    //渲染數據
    total.value = result.data.total;
    articles.value = result.data.items;

    //給文章分類從代號變成名稱
    for (let i = 0; i < articles.value.length; i++) {
        let article = articles.value[i];
        for (let j = 0; j < categorys.value.length; j++) {
            if (article.categoryId == categorys.value[j].id) {
                article.categoryName = categorys.value[j].categoryName;
            }
        }
    }
}

articleCategoryList();
articleList();

import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css'
import { Plus } from '@element-plus/icons-vue'
//控制抽屜是否顯示
const visibleDrawer = ref(false)
//新增表單資料模型
const articleModel = ref({
    title: '',
    categoryId: '',
    coverImg: '',
    content: '',
    state: ''
})
//導入token
import { useTokenStore } from '@/stores/token.js';
const tokenStore = useTokenStore();

//上傳成功的回調函數
const updateSuccess = (result) => {
    articleModel.value.coverImg = result.data;
    console.log(result.data);
}

//添加文章
import { ElMessage, ElMessageBox } from 'element-plus';
const addArticle = async (clickState) => {
    articleModel.value.state = clickState;

    let result = await articleAddService(articleModel.value);
    ElMessage.success(result.msg ? result.msg : '添加成功');

    visibleDrawer.value = false;
    articleList();
}
//定義彈跳視窗標題
const title = ref('')
//展示編輯彈跳視窗
const showDialog = (row) => {
    visibleDrawer.value = true;
    title.value = '編輯文章'
    // 數據拷貝
    articleModel.value.title = row.title;
    articleModel.value.categoryId = row.categoryId;
    articleModel.value.coverImg = row.coverImg;
    articleModel.value.content = row.content;
    articleModel.value.state = row.state;
    //增加id屬性,後臺編輯數據需要
    articleModel.value.id = row.id
}
//刪除文章
const deleteArticle = (row) => {
    ElMessageBox.confirm(
        '你確認要將該分類刪除嗎?',
        '提示',
        {
            confirmButtonText: '確認',
            cancelButtonText: '取消',
            type: 'warning',
        }
    )
        .then(async () => {
            let result = await articleDeleteService(row.id);
            ElMessage({
                type: 'success',
                message: '刪除成功',
            })
            //重新載入列表
            articleList();

        })
        .catch(() => {
            ElMessage({
                type: 'info',
                message: '刪除取消',
            })
        })
}

//編輯文章
const updateArticle = async (clickState) => {
    articleModel.value.state = clickState;
    let result = await articleUpdateService(articleModel.value);
    ElMessage.success(result.msg ? result.msg : '編輯成功')
    //重新載入列表
    articleList();
    //關閉編輯視窗
    visibleDrawer.value = false;
}

//清空表單
const clearData = () => {
    // 數據拷貝
    articleModel.value.title = '';
    articleModel.value.categoryId = '';
    articleModel.value.coverImg = '';
    articleModel.value.content = '';
    articleModel.value.state = '';
}

</script>
<template>
    <el-card class="page-container">
        <template #header>
            <div class="header">
                <span>文章管理</span>
                <div class="extra">
                    <el-button type="primary" @click="clearData(); visibleDrawer = true; title = '新增文章'">新增文章</el-button>
                </div>
            </div>
        </template>
        <!-- 搜尋表單 -->
        <el-form inline class="page-form">
            <el-form-item label="文章分類：">
                <el-select placeholder="請選擇" v-model="categoryId">
                    <el-option v-for="c in categorys" :key="c.id" :label="c.categoryName" :value="c.id">
                    </el-option>
                </el-select>
            </el-form-item>

            <el-form-item label="發佈狀態：">
                <el-select placeholder="請選擇" v-model="state">
                    <el-option label="已發佈" value="已發佈"></el-option>
                    <el-option label="草稿" value="草稿"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="articleList()">搜尋</el-button>
                <el-button @click="categoryId = ''; state = ''">重置</el-button>
            </el-form-item>
        </el-form>
        <!-- 文章列表 -->
        <el-table :data="articles" style="width: 100%">
            <el-table-column label="文章標題" width="250" prop="title"></el-table-column>
            <el-table-column label="文章封面" width="180">
                <template #default="{ row }">
                    <img v-if="row.coverImg" :src="row.coverImg" alt="封面"
                        style="width: 120px; height: 80px; object-fit: cover;">
                    <span v-else>無</span>
                </template>
            </el-table-column>
            <el-table-column label="分類" width="180" prop="categoryName"></el-table-column>
            <el-table-column label="發表時間" prop="createTime"> </el-table-column>
            <el-table-column label="狀態" prop="state"></el-table-column>
            <el-table-column label="操作" width="100">
                <template #default="{ row }">
                    <el-button :icon="Edit" circle plain type="primary" @click="clearData(); showDialog(row)"></el-button>
                    <el-button :icon="Delete" circle plain type="danger" @click="deleteArticle(row)"></el-button>
                </template>
            </el-table-column>
            <template #empty>
                <el-empty description="沒有數據" />
            </template>
        </el-table>
        <!-- 分頁條 -->
        <el-pagination v-model:current-page="pageNum" v-model:page-size="pageSize" :page-sizes="[3, 5, 10, 15]"
            layout="jumper, total, sizes, prev, pager, next" background :total="total" @size-change="onSizeChange"
            @current-change="onCurrentChange" style="margin-top: 20px; justify-content: flex-end" />

        <el-drawer v-model="visibleDrawer" :title="title" direction="rtl" size="50%">
            <!-- 新增文章表單 -->
            <el-form :model="articleModel" label-width="100px">
                <el-form-item label="文章標題">
                    <el-input v-model="articleModel.title" placeholder="請輸入標題"></el-input>
                </el-form-item>
                <el-form-item label="文章分類">
                    <el-select placeholder="請選擇" v-model="articleModel.categoryId">
                        <el-option v-for="c in categorys" :key="c.id" :label="c.categoryName" :value="c.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="文章封面">
                    <el-upload class="avatar-uploader" :auto-upload="true" :show-file-list="false" action="/api/upload"
                        name="file" :headers="{ 'Authorization': tokenStore.token }" :on-success="updateSuccess">
                        <img v-if="articleModel.coverImg" :src="articleModel.coverImg" class="avatar" />
                        <el-icon v-else class="avatar-uploader-icon">
                            <Plus />
                        </el-icon>
                    </el-upload>
                </el-form-item>
                <el-form-item label="文章內容">
                    <div class="editor"><quill-editor theme="snow" v-model:content="articleModel.content"
                            contentType="html">
                        </quill-editor></div>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary"
                        @click="title == '新增文章' ? addArticle('已發佈') : updateArticle('已發佈')">發佈</el-button>
                    <el-button type="info" @click="title == '新增文章' ? addArticle('草稿') : updateArticle('草稿')">草稿</el-button>
                </el-form-item>
            </el-form>
        </el-drawer>
    </el-card>
</template>
<style lang="scss" scoped>
.page-container {
    min-height: 100%;
    box-sizing: border-box;
    background-color: rgb(255, 255, 255);

    /* 透明的黑色背景 */
   

    .header {
        display: flex;
        align-items: center;
        justify-content: space-between;

    }

    /* 將 el-card 的背景色設定為黑色 */

}

.page-form {
    background-color: #ffffff; /* 黑色背景 */
}

.page-form .el-form-item__label,
.page-form .el-select,
.page-form .el-button {
    color: #fff; /* 白色字體顏色 */
}



.avatar-uploader {
    :deep() {
        .avatar {
            width: 178px;
            height: 178px;
            display: block;

        }

        .el-upload {
            border: 1px dashed var(--el-border-color);
            border-radius: 6px;
            cursor: pointer;
            position: relative;
            overflow: hidden;
            transition: var(--el-transition-duration-fast);

        }

        .el-upload:hover {
            border-color: var(--el-color-primary);

        }

        .el-icon.avatar-uploader-icon {
            font-size: 28px;
            color: #8c939d;
            width: 178px;
            height: 178px;
            text-align: center;

        }
    }
}

.editor {
    width: 100%;

    :deep(.ql-editor) {
        min-height: 200px;
    }

}</style>