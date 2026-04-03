<template>
    <el-dialog
        :title="isEdit?'编辑文章':'新增文章详情'"
        v-model="dialogVisble"
        width="50%"
        @close="handleClose"
    >
    <el-form :model="formData" ref="FormRef" :rules="rules" label-width="120px">
        <el-form-item label="文章标题" prop="title">
            <el-input v-model="formData.title" placeholder="请输入文章标题" maxlength="200"  show-word-limit clearable />
        </el-form-item>
        <el-form-item label="分类" prop="categoryId">
            <el-select v-model="formData.categoryId" placeholder="请选择分类">
                <el-option v-for="item in props.categories" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
        </el-form-item>
        <el-form-item label="文章摘要" prop="summary">
            <el-input type="textarea" v-model="formData.summary" placeholder="请输入文章摘要(可选)" maxlength="1000"  show-word-limit :rows="4" />
        </el-form-item>
        <el-form-item label="标签" prop="tags">
            <el-select v-model="formData.tagArray" placeholder="请输入文章标签(逗号分隔)" multiple filterable allow-create width="100%" >
                <el-option v-for="tag in commonTags" :key="tag" :label="tag" :value="tag" />
            </el-select>
        </el-form-item>
        <el-form-item label="封面图片" >
            <div class="cover-upload">
                <el-upload
                    class="avatar-uploader"
                    action="#"
                    :before-upload="beforeUpload"
                    :http-request="handleUploadRequest"
                    accept="image/*"
                    :show-file-list="false"
                >
                    <div v-if="!imgUrl" class="cover-placeholder">
                        <p>点击上传封面图片</p>
                    </div>    
                    <img v-else :src="imgUrl" alt="封面图片" class="cover-image" />               
                </el-upload>
                    <div v-if="imgUrl" class="cover-remove">
                        <el-button type="danger" size="mini" @click="handleRemove">移除封面</el-button>
                    </div>
            </div>
        </el-form-item>
        <el-form-item label="文章内容" prop="content">
            <RichTextEditor 
            v-model="formData.content" 
            placeholder="请输入文章内容"
            maxCharCount="5000"
            @change="handleContentChange"
            @create="handleEditorCreate"
            min-height="400px"
            />
        </el-form-item>
    </el-form>
    <div v-if="btnPreview" class="preview-container">
        <h3>预览效果</h3>
        <div v-html="formData.content"></div>
    </div>
    <template #footer>
        <el-button  @click="btnPreview=!btnPreview">{{btnPreview?'隐藏预览':'预览效果'}}</el-button>
        <el-button  @click="handleClose">取消</el-button>
        <el-button type="primary" @click="handleSubmit" :loading="loading">{{isEdit?'更新文章':'创建文章'}}</el-button>
    </template>
    </el-dialog>
</template>

<script setup>
import { ref, computed, reactive,nextTick,watch } from 'vue'
import { ElMessage } from 'element-plus'
import { uploadFile,createArticle,updateArticle } from '@/api/admin'
import { filebaseURL } from '@/config/index'
import RichTextEditor from '@/components/RichTextEditor.vue'

const articleId = ref(null)
const uploadBusinessId = ref(null)

const handleClose=()=>{
    FormRef.value.resetFields()
    articleId.value = null
    uploadBusinessId.value = null
    handleRemove()
    formData.tagArray=[]
    emit('update:modelValue',false)
}
const props=defineProps({
    modelValue:{
        type:Boolean,
        default:false
    },
    categories:{
        type:Array,
        default:() => []
    },
    Article:{
        type:Object,
        default:null
    }
})
const emit=defineEmits(['update:modelValue','success'])

const isEdit=computed(()=>{
    return !!props.Article?.id
})



//监听编辑文章
watch(()=>props.Article,async (newVal,oldVal)=>{
    if(newVal){
        nextTick(()=>{
            Object.assign(formData,newVal)
            //使用现有id
            //businessid.value=newVal.id
            articleId.value = newVal.id
            uploadBusinessId.value = newVal.id
            //封面Url
            imgUrl.value=filebaseURL+newVal.coverImage
        })

    }
})

const dialogVisble=computed({
    get(){
        return props.modelValue
    },
    set(val){
        emit('update:modelValue',val)
    }
})


// 表单数据
const formData=reactive({    
    "title": "",
    "content": "",
    "coverImage": "",
    "categoryId": 1,
    "summary": "",
    "tags": "",
    "id": ""
})
const rules=reactive({
    title: [
        { required: true, message: '请输入文章标题', trigger: 'blur' }
    ],
    categoryId: [
        { required: true, message: '请选择分类', trigger: 'change' }
    ],
    summary: [
        { required: false, message: '请输入文章摘要', trigger: 'blur' }
    ],
    content: [
        { required: true, message: '请输入文章内容', trigger: 'blur' },
        { max: 5000, message: '文章内容长度必须5000以内', trigger: 'blur' }
    ],
})

const commonTags = [
  '情绪管理', '焦虑', '抑郁', '压力', '睡眠', 
  '冥想', '正念', '放松', '心理健康', '自我成长',
  '人际关系', '工作压力', '学习方法', '生活技巧'
]
//上传
const imgUrl=ref('')
const beforeUpload=(file)=>{
    const isImage=file.type.startsWith('image/')
    const isLt5MB=file.size/1024/1024<5
    if(!isImage){
        ElMessage.error('请上传图片文件')
        return false
    }
    if(!isLt5MB){
        ElMessage.error('图片大小不能超过5MB')
        return false
    }
    return true
}
const handleUploadRequest= async ({file})=>{
    if (isEdit.value) {
        uploadBusinessId.value = articleId.value
    } else if (!uploadBusinessId.value) {
        uploadBusinessId.value = crypto.randomUUID()
    }
   const fileRes=await uploadFile(file,{
    businessId: uploadBusinessId.value    })
    //拼接完整图片地址
    imgUrl.value=filebaseURL+fileRes.filePath
    formData.coverImage=fileRes.filePath


}
//移除封面
const handleRemove=()=>{
    imgUrl.value=''
    formData.coverImage=''
}
    
const handleContentChange=(data)=>{
    formData.content=data.html
    
}

const editorInstance=ref(null)
const handleEditorCreate=(editor)=>{
    editorInstance.value=editor
    //编辑
    if(formData.content && editor){
        nextTick(()=>{
            editor.setHtml(formData.content)
        })
    }
}

const btnPreview=ref(false)
//提交
const loading=ref(false)
//提交表单
const FormRef=ref()
const handleSubmit=()=>{
    FormRef.value.validate((valid,fields)=>{
        if(valid){
            loading.value=true
            console.log(formData,'formData')
            console.log(fields,'fields')
            const submitdata={
                ...formData,
                tags:formData.tagArray.join(',')
            }
            delete submitdata.tagArray
           if(!isEdit.value){
                createArticle(submitdata).then(res=>{
                    loading.value=false
                    ElMessage.success('创建成功')
                    emit('success')
                })
           }else{
                updateArticle(articleId.value,submitdata).then(res=>{
                    loading.value=false
                    ElMessage.success('更新成功')
                    emit('success')
                })
           }
        }
    })
}


</script>

<style lang="scss" scoped>
.cover-placeholder{
    width: 200px;
    height: 120px;
    display: flex;
    flex-direction: column; //????
    align-items: center;
    justify-content: center;
    color: #8b949e;
    background-color: #f6f8fa;
}
.cover-image{
    width: 200px;
    height: 120px;
    display: block;
}
</style>