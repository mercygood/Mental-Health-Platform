<template>
    <div>
        <PageHead title="知识文章">
            <template #buttons>
                <el-button type="primary" @click="handleEdit({})">新增</el-button>
            </template>
        </PageHead>
        <TableSearch :formItem="formItem"  @search="handSearch" />
        <el-table :data="tableData" style="width: 100% ;margin-top: 25px">
            <el-table-column prop="title" label="文章标题" fixed="left">
                <template #default="scope">
                    <div style="display: flex; align-items: center;">
                        <el-icon><timer /></el-icon>
                        <span>{{scope.row.title}}</span>
                    </div>
                </template>
            </el-table-column>
            <el-table-column prop="categoryId" label="分类" width="200" >
                <template #default="scope">
                    <div style="display: flex; align-items: center;">
                        <el-icon><timer /></el-icon>
                        <span>{{categoryMap[scope.row.categoryId]}}</span>
                    </div>
                </template>
            </el-table-column>
            <el-table-column prop="authorName" label="作者" width="150" />
            <el-table-column prop="readCount" label="阅读量" width="150" />
            <el-table-column prop="updatedAt" label="更新时间" width="150" />
            <el-table-column  label="操作" width="240"  fixed="right">
                <template #default="scope">
                    <el-button text type="primary" @click="handleEdit(scope.row)" >编辑</el-button>
                    <el-button @click="handlePublish(scope.row)" v-if="scope.row.status==0 || scope.row.status==2" text type="success" >发布</el-button>
                    <el-button @click="handleDown(scope.row)" v-if="scope.row.status==1" text type="warning" >下线</el-button>
                    <el-button @click="handleDelete(scope.row)" text type="danger" >删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination
            style="margin-top: 25px"
            :page-size="pagination.size"
            layout=" prev, pager, next"
            :total="pagination.total"
            @current-change="handleChange"
        />
        <ArticleDialog v-model:modelValue="DialogVisvble" :categories="categories" @success="handleSuccess" :Article="currentArticle" />
    </div>
</template>

<script setup>
import { reactive ,ref} from 'vue'
import { onMounted } from 'vue'
import TableSearch from '../components/TableSearch.vue';
import PageHead from '../components/PageHead.vue';
import { CategoryTree ,articlePage,changeArticlesStatus} from '../api/admin'
import ArticleDialog from '../components/ArticleDialog.vue'
import { getArticleDetail,deleteArticle } from '../api/admin'
import {ElMessageBox} from 'element-plus'
import {ElMessage} from 'element-plus'


    const formItem=[
        {
            comp:'input',prop:'title',label:'文章标题',placeholder:'请输入文章标题'
        },
        {
            comp:'select',prop:'categoryID',label:'分类',placeholder:'请选择分类'
        },
        {
            comp:'select',prop:'status',label:'状态',placeholder:'请输入文章内容',options:[
                {
                    label:'已发布',
                    value:1
                },
                {
                    label:'已下线',
                    value:2
                },                
                {
                    label:'草稿',
                    value:0
                }
            ]
            }
    ]

    //分页参数
    const pagination=reactive({
        currentPage:1,
        size:10,
        total:0
    })
//列表数据
const tableData=ref([])
const searchFormData = ref({})

//新增和编辑文章
const DialogVisvble=ref(false)


const handSearch = async (formData)=> {
    console.log(formData);
    searchFormData.value=formData
    const params={
        ...pagination,
        ...formData
    }
    console.log('发送的参数:', params);
    const { records ,total }=await articlePage(params)
    tableData.value=records
    pagination.total=total
}
const handleChange=(page)=>{
    pagination.currentPage=page
    handSearch(searchFormData.value)
}
//分类映射
const categories=ref([])
//分类列表
const categoryMap=reactive([])

//新增和编辑文章成功后刷新列表
const handleSuccess=()=>{
    DialogVisvble.value=false
    handSearch(searchFormData.value)
}

onMounted(async()=>{
    //获取分类树
    const data=await CategoryTree()
    categories.value=data.map(item =>{
        categoryMap[item.id]=item.categoryName
        return {
            label:item.categoryName,
            value:item.id
        }
    
    })
    console.log( '分类列表:',categories.value);
    formItem[1].options=categories.value

    //获取文章列表
handSearch({})
}
)
const currentArticle=ref(null)
const handleEdit=(row)=>{
    if(!row.id){
        currentArticle.value=null
        DialogVisvble.value=true
    }else{
    getArticleDetail(row.id).then(res=>{
        console.log(res);
        currentArticle.value=res
        DialogVisvble.value=true
        })
    }
}
//发布文章
const handlePublish=(row)=>{
    ElMessageBox.confirm(`确认发布文章${row.title}吗？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'info'
    }).then(() => {
        changeArticlesStatus(row.id,{status:1}).then(res=>{
        console.log(res);
        ElMessage.success('发布成功')
        handSearch(searchFormData.value)
    })
    })
}
//下线文章
const handleDown=(row)=>{
    ElMessageBox.confirm(`确认下线文章${row.title}吗？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'info'
    }).then(() => {
        changeArticlesStatus(row.id,{status:2}).then(res=>{
        console.log(res);
        ElMessage.success('下线成功')
        handSearch(searchFormData.value)
    })
})
}
//删除文章
const handleDelete=(row)=>{
    ElMessageBox.confirm(`确认删除文章${row.title}吗？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'danger'
    }).then(() => {
        deleteArticle(row.id).then(res=>{
        console.log(res);
        ElMessage.success('删除成功')
        handSearch(searchFormData.value)
    })
})
}



</script>