<template>
	<view>
		<text>停车app</text>
		<text>用户名{{username}}</text>
		<button @click="LogOut">退出登录</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username:'',
				password:'',
			}
		},
		onLoad(e) {                              //尝试获取本地账号密码，登录后记录token以及userId
			console.log("index开始加载");
			var that=this;
			uni.getStorage({
				key:'username',
				fail() {
					console.log('username获取失败');
					uni.redirectTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					console.log('username获取成功');
					that.username=JSON.parse(res.data)
					//console.log(that.username)
					uni.getStorage({
						key:'password',
						fail() {
							console.log('password获取失败');
							uni.redirectTo({
								url:'/pages/Login/Login',
							})
						},
						success:function(res){
							console.log('password获取成功');
							that.password=JSON.parse(res.data)
							//console.log(that.password)
							
							//自动登录
							let p={};
							let key='username';
							let value=that.username;
							p[key]=value;
							key='password';
							value=that.password;
							p[key]=value;
							//console.log(p);
							uni.request({
								url:'http://101.34.159.132:8080/login',
								method:'POST',
								data:JSON.stringify(p),
								success: res => {
									console.log(p);
									console.log(res);
									if(res.data.message=="登录成功"){
										console.log('自动登录成功');
										uni.setStorage({
											key:'token',
											data:JSON.stringify(res.data.data.token),
											complete(data) {
												console.log('设置token完成')
												//console.log(data)
												uni.request({//获取用户信息
													url:'http://101.34.159.132:8080/user/findUserByUserName/'+that.username,
													method:'GET',
													header:{token:res.data.data.token},//无语
													success: res => {
														//console.log(res);
														if(res.data.message=='操作成功'){
															console.log("获取用户信息成功");
															uni.setStorage({
																key:'userId',
																data:res.data.data.userId,
																success(data) {
																	console.log('设置userId成功');
																	console.log('即将跳转');
																	uni.switchTab({
																		url:'/pages/Map/Map',
																	})
																},
																fail() {
																	console.log('设置userId失败');
																	//uni.clearStorage();
																	uni.redirectTo({
																		url:'/pages/Login/Login',
																	})
																}
															})
														}else{
															console.log('获取用户信息失败');
															//uni.clearStorage();
															uni.redirectTo({
																url:'/pages/Login/Login',
															})
														}
													},
													fail() {
														console.log('获取用户信息失败2.0');
														//uni.clearStorage();
														uni.redirectTo({
															url:'/pages/Login/Login',
														})
													},
												})
											}
										})
							
									}
									else{
										console.log('自动登录失败');
										//uni.clearStorage();
										uni.redirectTo({
											url:'/pages/Login/Login',
										})
									};
									
								},
								fail() {
									console.log('自动登录失败2.0');
									//uni.clearStorage();
									uni.redirectTo({
										url:'/pages/Login/Login',
									})
								}
							})
						}
					})
				}
			})
			
		},
		methods: {
			LogOut(){
				var that=this;
				uni.request({
					url:'http://101.34.159.132:8080/logOut',
					method:'GET',
					header:{token:that.token},
					success(res) {
						console.log('退出登录成功');
						uni.clearStorage();
						uni.redirectTo({
							url:'/pages/Login/Login',
						})
					},
					fail() {
						console.log('退出登录失败');
					}
				})
			}
		}
	}
</script>

<style>

</style>