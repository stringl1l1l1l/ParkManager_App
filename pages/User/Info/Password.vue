<!--  -->
<template>
	<view>
		<uni-nav-bar left-icon="left" leftText="返回" rightText="保存" title="密码修改" @clickLeft="clickLeft" @clickRight="clickRight"/>
		<uni-forms ref="form" :modelValue="formData" :rules="rules">
			<uni-forms-item name="old_password">
				<uni-easyinput  v-model="formData.old_password" placeholder="请输入旧密码" type="password"/>
			</uni-forms-item>
			<uni-forms-item name="new_password">
				<uni-easyinput  v-model="formData.new_password" placeholder="请输入新密码" type="password"/>
			</uni-forms-item>
			<uni-forms-item name="new_password2">
				<uni-easyinput  v-model="formData.new_password2" placeholder="确认新密码" type="password" />
			</uni-forms-item>
		</uni-forms>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				token:'',
				userId:'0',
				password:'',
				formData:{
					old_password:'',
					new_password:'',
					new_password2:'',
				},
				rules:{
					old_password:{
						rules:[
							{
								required:true,
								errorMessage:"请输入旧密码",
							}
						]
					},
					new_password:{
						rules:[
							{
								required:true,
								errorMessage:"请输入新密码",
							},
							{
								minLength:6,
								errorMessage:"密码应长于6个字符",
							},
							{
								maxLength:15,
								errorMessage:"密码不应长于15个字符"
							},
						]
					},
					new_password2:{
						rules:[
							{
								required:true,
								errorMessage:"请确认新密码",
							},
							{
								validateFunction:function(rule,value,data,callback){
									if(value!=data.new_password)
									{
										callback('两次密码输入不一致')
									}
									return true
								}
							}
						]
					},
				}
			}
		},
		onLoad() {
			var that=this;
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
				key:'userId',
				fail() {
					console.log('userId获取失败');
					uni.switchTab({
						url:'pages/User/User'
					})
				},
				success:function(res){
					console.log('userId获取成功');
					that.userId=JSON.parse(res.data)
					console.log(that.userId)
				}
			})
			uni.getStorage({
				key:'token',
				fail() {
					console.log('token获取失败');
					uni.switchTab({
						url:'pages/User/User'
					})
				},
				success:function(res){
					console.log('token获取成功');
					that.token=JSON.parse(res.data)
				}
			})
		},
		methods: {
			clickLeft(){
				uni.navigateBack({
					delta: 1,
				})
			},
			clickRight(){
				this.$refs.form.setRules(this.rules)
				var that=this;
				this.$refs.form.validate().then(res=>{
					if(that.password!=res.old_password){
						uni.showToast({
							title:'旧密码不正确',
							icon:'error',
						})
						return;
					}
					
					console.log("ok");
					res["userId"]=that.userId;
					res["password"]=res.new_password;
					console.log(res);
					uni.request({
						url:'http://101.34.159.132:8080/user/updateUserById',
						data:res,
						header:{token:that.token},
						method:'PUT',
						success(r) {
							console.log(r);
							if(r.data.message=="操作成功")
							{
								uni.showToast({
									title:'修改成功',
									complete() {
										uni.setStorage({
											key:'password',
											data:JSON.stringify(res.password),
											success(data) {
												console.log('设置password完成')
												setTimeout(()=>{that.clickLeft();},500)
											}
										})
									}
								});			
							}
							else{
								uni.showToast({
									title:r.data.message,
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

</style>
