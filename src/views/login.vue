<template>
    <div class="container">
        <div class="title">
            <div class="back-home">
                <el-icon><Back /></el-icon>
                <p>返回首页</p>
            </div>
            <div class="title-text">
                <h2>登录您的账户</h2>
                <p>请输入您的登录信息</p>
            </div>
        </div>
        <div class="form-content">
            <el-form ref="ruleFormRef" :model="formData" :rules="rules" label-position="top">
                <el-form-item label="用户名或邮箱" prop="username">
                    <el-input v-model="formData.username" size="large" placeholder="请输入用户名或邮箱"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="formData.password" size="large" type="password" show-password placeholder="请输入密码"></el-input>
                </el-form-item>
                <el-button class="btn" type="primary" size="large" @click="submitForm(ruleFormRef)">登录</el-button>
            </el-form>
            <div class="footer">
                <p>还没有账户？<router-link to="/auth/register">去注册</router-link></p>
            </div>
        </div>
    </div>
</template>

<script setup>
import { reactive, ref } from 'vue';
import { login } from '../api/admin'
import { useRouter } from 'vue-router'

const router = useRouter()

const ruleFormRef=ref()

const formData=reactive({
    username :'' ,
    password :''
})
const rules=reactive({
    username:[
        {required:true,message:'请输入用户名',trigger:'blur'}
    ],
    password:[
        {required:true,message:'请输入用密码',trigger:'blur'}
    ]
})

//denglu
const submitForm=async(formEL)=>{
    if(!formEL) return
    await formEL.validate((valid,fields)=>{
        if(valid){
            login(formData).then(data =>{
                //判断token是否存在
                if(!data.token){
                    return console.error('登录失败')
                }
                // 登录成功，将token存储到localStorage
                localStorage.setItem('userInfo', JSON.stringify(data.userInfo))
                localStorage.setItem('token', data.token)
                //根据用户角色跳转
                if(data.userInfo.userType === 2){
                    router.push('/back/dashboard')
                }else{
                    router.push('/')
                }
            })
        }
    })
}
// const submitForm = async () => {
//   if (!ruleFormRef.value) return
//   await ruleFormRef.value.validate((valid, fields) => {
//     if (valid) {
//       console.log('校验通过',fields)
//     }
//   })
// }


</script>

<style lang="scss" scoped>

    .container{
        width: 384px;
        .title{
            .back-home{
                margin-bottom: 60px;
            }            
        }
        .title-text{
            text-align: center;
            h2{
                font-size: 36px;
                margin-bottom: 10px;
            }
            p{
                font-size: 20px;
            }
        }
        .form-content{
            margin-top: 30px;
            .btn{
               margin-top: 40px;
               width: 100%; 
            }
            .footer{
                padding: 30px;
                text-align: center;
            }
        }
    }

</style>