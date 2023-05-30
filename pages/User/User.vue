<template>
	<view>
		<uni-list border-full="true">
			<uni-list-item title="用户id:" :rightText="userId"/>
			<uni-list-item title="用户名:" :rightText="username"/>
			<uni-list-item title="车牌号:" :rightText="carNum" showArrow="true" clickable to="Info/CarNum"/> 
			<uni-list-item title="手机号:" :rightText="phoneNum" showArrow="true" clickable to="Info/PhoneNum"/> 
			<uni-list-item title="注册时间:" :rightText="registerTime"/>
			<uni-list-item title="密码:" rightText="******" showArrow="true" clickable to="Info/Password"/>
		</uni-list>
		<button @click="LogOut" >退出登录</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username:'null',
				token:'',
				userId:'0',
				carNum:'未填写',
				phoneNum:'未填写',
				registerTime:'null',
				password:'',
			}
		},
		onReady() {
			uni.setNavigationBarTitle({
			    title: '个人信息'
			});
		},
		onShow() {
			var that=this;
			uni.getStorage({
				key:'username',
				fail() {
					uni.navigateTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					that.username=JSON.parse(res.data);
					console.log(that.username);
				}
			})
			uni.getStorage({
				key:'password',
				fail() {
					uni.navigateTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					that.password=JSON.parse(res.data);
					console.log(that.password);
				}
			})
			uni.getStorage({
				key:'token',
				fail() {
					uni.navigateTo({
						url:'/pages/Login/Login',
					})
				},
				success:function(res){
					that.token=JSON.parse(res.data);
					//console.log(that.token);
					uni.request({//获取用户信息
						url:'http://101.34.159.132:8080/user/findUserByUserName/'+that.username,
						method:'GET',
						header:{token:that.token},
						success: res => {
							console.log(res);
							if(res.data.message=='操作成功'){
								console.log("获取用户信息成功");
								that.carNum=res.data.data.carNum==null?"未填写":res.data.data.carNum;
								that.userId=res.data.data.userId.toString();
								// that.password=res.data.data.password;
								that.registerTime= new Date(res.data.data.registerTime).toDateString();
								that.phoneNum=res.data.data.phoneNum==null?"未填写":res.data.data.phoneNum;
								uni.setStorage({
									key:'userId',
									data:res.data.data.userId,
									complete(data) {
										console.log('设置userId完成');
									},
									fail() {
										console.log('设置userId失败');
									}
								})
							}else{
								uni.showToast({
									title:'网络错误',
									icon:'error'
								});
							}
						},
						fail() {
							cuni.showToast({
								title:'网络错误',
								icon:'error'
							});
						},
					})
					
				},
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
	.user_row{
		display: flex;
		width: 100%;
		justify-content: space-between;
	}

</style>