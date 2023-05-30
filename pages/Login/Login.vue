<template>
	<view>
		<image src="../../static/parking_PNG7.png" class="Logo"></image>
		
		<uni-forms ref="form" :modelValue="formData" style="margin: 50rpx;">
			<uni-forms-item name="username">
				<uni-easyinput  v-model="formData.username" placeholder="请输入用户名"/>
			</uni-forms-item>
			<uni-forms-item name="password">
				<uni-easyinput  v-model="formData.password" type="password" placeholder="请输入密码"/>
			</uni-forms-item>
		</uni-forms>
		<button class="button" @click="submit">登录</button>
		<navigator url="../Register/Register"><button class="button">注册</button></navigator>
		
		<image src="../../static/background.jpeg"style="margin: 40rpx;"></image>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				formData:{
					username:'',
					password:'',
				},
			}
		},
		onReady() {
			uni.setNavigationBarTitle({
			    title: '欢迎使用！'
			});
		},
		methods: {
			submit:function(){
				var that=this;
				this.$refs.form.validate().then(res=>{
					// console.log(res);
					// console.log(JSON.stringify(res));
					uni.request({
						url:'http://101.34.159.132:8080/login',
						method:'POST',
						data:JSON.stringify(res),
						success: r => {
							console.log(r);
							if(r.data.message=='登录成功'){
								uni.setStorage({
									key:'username',
									data:JSON.stringify(res.username),
									success(data) {
										console.log('设置username完成')
										//console.log(data)
									}
								})
								uni.setStorage({
									key:'password',
									data:JSON.stringify(res.password),
									success(data) {
										console.log('设置password完成')
										//console.log(data)
									}
								})
								uni.setStorage({
									key:'token',
									data:JSON.stringify(r.data.data.token),
									success(data) {
										console.log('设置token完成')
										uni.request({//获取用户信息
											url:'http://101.34.159.132:8080/user/findUserByUserName/'+res.username,
											method:'GET',
											header:{token:r.data.data.token},//无语
											success: res => {       //这里res是局部变量
												console.log(res);
												if(res.data.message=='操作成功'){
													console.log("获取userId成功");
													uni.setStorage({
														key:'userId',
														data:res.data.data.userId,
														success(data) {
															console.log('设置userId成功');
															uni.showToast({
																title: '登录成功'
															});
															console.log('即将跳转');
															uni.switchTab({
																url:'/pages/Map/Map',
															})
														},
														fail() {
															uni.showToast({
																title: '设置userId失败',
																icon: 'error'
															});
														}
													})
												}else{
													uni.showToast({
														title: '获取userId失败',
														icon: 'error'
													});
												}
											},
											fail() {
												uni.showToast({
													title: '获取userId失败',
													icon: 'error'
												});
											},
										})
									}
								})
							}
							else{
								uni.showToast({
									title: '请检查用户名或密码',
									icon:"none"
								});
							}
						},
						fail() {
							uni.showToast({
								title:"网络错误",
								icon: 'error'
							})
						}
					})
				}).catch(err=>{
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
		font:'Gill Sans', 
	}
	.Logo{
		 margin-top: 50rpx;
		 margin-left: 300rpx;
		
		 width: 125rpx;
		 height: 125rpx;
	}
	
</style>