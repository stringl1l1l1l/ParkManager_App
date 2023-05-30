<template>
	<view>
		<uni-nav-bar left-icon="left" leftText="返回" rightText="保存" title="车牌修改" @clickLeft="clickLeft" @clickRight="clickRight"/>
		<uni-forms ref="form" :modelValue="formData" :rules="rules">
			<uni-forms-item name="carNum">
				<uni-easyinput  v-model="formData.carNum" placeholder="请输入车牌号" />
			</uni-forms-item>
		</uni-forms>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				userId:'',
				token:'',
				formData:{
					carNum:'',
				},
				rules:{
					carNum:{
						rules:[
							{
								required:true,
								errorMessage:"请输入车牌号",
							}
						]
					},
				}
			}
		},
		onLoad() {
			var that=this;
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
					//console.log(that.userId)
				}
			});
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
				var that=this;
				this.$refs.form.validate().then(res=>{
					console.log("ok");
					res["userId"]=that.userId;
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
										setTimeout(()=>{that.clickLeft();},500)
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
