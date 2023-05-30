<template>
	<view>
		<uni-forms ref="form" :modelValue="formData" :rules="rules">
			<uni-forms-item style="margin: 10rpx;" name="username">
				<uni-easyinput  v-model="formData.username" placeholder="请输入用户名"  />
			</uni-forms-item>
			<uni-forms-item style="margin: 10rpx;" name="password">
				<uni-easyinput  v-model="formData.password" placeholder="请输入密码" type="password"/>
			</uni-forms-item>
			<uni-forms-item style="margin: 10rpx;"  name="passwordConfirm">
				<uni-easyinput  v-model="formData.passwordConfirm" placeholder="确认密码" type="password"/>
			</uni-forms-item>
		</uni-forms>
		<button @click="submit" class="button">注册</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				token:'',
				formData:{
					username:'',
					password:'',
					passwordConfirm:'',
				},
				rules:{
					username:{
						rules:[
							{
								required:true,
								errorMessage:"请输入用户名",
							},
							{
								maxLength:15,
								errorMessage:"用户名不能超过15个字符",
							},
							{
								minLength:1,
								errorMessage:"用户名不应少于1个字符"
							}
						]
					},
					password:{
						rules:[
							{
								required:true,
								errorMessage:"请输入密码",
							},
							{
								minLength:6,
								errorMessage:"密码应长于6个字符",
							},
							{
								maxLength:15,
								errorMessage:"密码不应长于15个字符"
							}
						]
					},
					passwordConfirm:{
						rules:[
							{
								required:true,
								errorMessage:"请确认密码",
							},
							{
								validateFunction:function(rule,value,data,callback){
									if(value != data.password)
									{
										callback('两次密码输入不一致')
									}
									return true
								},
							}
						]
						
					},
				}
			}
		},
		onReady() {
			this.$refs.form.setRules(this.rules)
			uni.setNavigationBarTitle({
				title: '用户注册'
			});
		},
		methods: {
			submit:function(){
				this.$refs.form.setRules(this.rules)
				console.log("注册");
				this.$refs.form.validate().then(res=>{
					console.log('ok');
					uni.request({
						url:'http://101.34.159.132:8080/register',
						data:JSON.stringify(res),
						method:'POST',
						success(r) {
							console.log(r);
							if(r.data.code=="200")
							{
								if(r.data.message=="注册成功"){
									uni.showToast({
										title:'注册成功',//未实现注册后自动登录
										
										complete() {
											uni.redirectTo({
												url:'../Login/Login'
											});
										}
									});
										
								}else{
									uni.showToast({
										title:'用户名重复',
										icon:'error',
									});
								}					
							}
							else{
								uni.showToast({
									title:'注册失败',
									icon:'error',
								});
							}
						},
						fail(){
							uni.showToast({
								title:'网络错误',
								icon:'error'
							});
						}
				 	})
				}
				).catch(err=>{
					console.log("no");
					console.log(err);
				})
			},
		}
	}
</script>

<style>
	.button{
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 50rpx;
		background-color: #55aaff;
		font:'Gill Sans';
	}
</style>