<template>
		<view style="background-color: #ebebeb; height: inherit;">
			<!-- <button @click="update_order">刷新订单</button> -->
			<div v-for="item in currentOrder" :key="item.id">
				<view class="order-item" >
					<view class="item-row">
						<text class="row-title" >
						{{parkingLot[myhash[item.parkingLotId]].description}}
						</text>
						<text :class="getSign(item.cancelFlag)">
						{{status[item.cancelFlag]}} 
						</text>
					</view>
					<view class="item-column">
						<text class="col-time">
						开始时间：{{ item.beginTime }}
						结束时间：{{item.endTime}} 
						</text>
						<text class="col-cost">
						消费￥{{item.cost}}元
						</text>
						<view class="col-wrapper">
							<text :class="getButton(item.cancelFlag)" @click="orderOperate" :data-info='item'>
							{{operate[item.cancelFlag]}}
							</text>
						</view>
					</view>
				</view>
			</div>
		</view>
	</template>

	<script>
		export default {
			data() {
				return {
					id:'',
					token:'',
					description:'',
					begTime:'',
					endTime:'',
					currentOrder:[],
					parkingLot:[],
					status:['已支付','已取消','已过期'],
					operate:['取消订单','删除记录','删除记录'],
					myhash:{}
				}
			},
			onReady() {
				uni.setNavigationBarTitle({
				    title: '我的订单'
				});
			},
			onPullDownRefresh() {
				this.update_order()
			},
			onShow(e) {
				var that = this
				uni.getStorage({
					key:'token',
					success:function(r)
					{
						that.token = r.data
						uni.getStorage({
							key:'userId',
							success(r) {
								that.id=r.data
								console.log(that.id)
								that.update_order()
							}
						})
					}
				})
				
				
			},
			methods: {
				getSign(flag){
					if(flag)
						return 'row-sign-1'
					else
						return 'row-sign-0'
				},
				getButton(flag){
					if(flag)
						return 'button-1'
					else
						return 'button-0'
				},
				getDateTime:function(sec){
					var D = new Date(sec)
					var res = D.getFullYear()+'/'+(D.getMonth()+1)+'/'+D.getDate()+' '
					res += D.getHours() > 9 ? D.getHours() : '0' + D.getHours()
					res += ':'
					res += D.getMinutes() > 9 ? D.getMinutes() : '0' + D.getMinutes()
					return res
				},
				update_order:function(){//刷新订单列表
					var that = this
					uni.request({
						url:'http://101.34.159.132:8080/parkingLot/findAllParkingLot',
						method:'GET',
						header:{token:JSON.parse(that.token)},
						success(res) {
							that.parkingLot=res.data.data
							console.log(that.parkingLot)
							that.myhash = {}
							for(var i=0;i<that.parkingLot.length;++i){
								that.myhash[that.parkingLot[i].parkingLotId] = i
							}
						}
					}),
					uni.request({
						url:'http://101.34.159.132:8080/order/findOrdersByUserId/'+that.id,
						method:'GET',
						header:{token:JSON.parse(that.token)},
						success(res) {
							console.log(res);
							
							that.currentOrder=res.data.data
							
							if(that.currentOrder.length==0)
							{
								uni.showToast({
									title:'您当前暂无订单',
									icon:'none'
								})
								return 
							}
							
							var cur_time=new Date().getTime();
							for(var i=0;i<that.currentOrder.length;++i){
								if(new Date(that.currentOrder[i].beginTime).getTime() <= cur_time){
									that.currentOrder[i].cancelFlag=2;
								}
								that.currentOrder[i].beginTime=that.getDateTime(new Date(that.currentOrder[i].beginTime).getTime()
																				+8*60*60*1000)
								that.currentOrder[i].endTime=that.getDateTime(new Date(that.currentOrder[i].endTime).getTime()
																				+8*60*60*1000)
								
							}
							
							that.currentOrder.sort(   //绿色优先，同色开始时间早的优先
							(a,b)=>{
								var aTime = new Date(a.beginTime).getTime()
								var bTime = new Date(b.beginTime).getTime()
								var res = 0
								if((a.cancelFlag && !b.cancelFlag) || (!a.cancelFlag && b.cancelFlag)){
									if(a.cancelFlag == 0) res = -1
									else if(b.cancelFlag == 0) res = 1
								}else{
									if(aTime < bTime) res = -1
									if(aTime > bTime) res = 1
								}
								return res
							}
							)
							
							
							
						},
						complete() {
							uni.stopPullDownRefresh();
						}
					})
				},
				orderOperate:function(e){
					console.log(e)
					var that = this
					var orderId = e.currentTarget.dataset.info.orderId
					if(e.currentTarget.dataset.info.cancelFlag==0)
					{
						uni.showModal({
								content:'确认要取消此订单吗',
								success(res)
								{
									if(res.confirm)
									{
										uni.request({
											url:'http://101.34.159.132:8080/order/cancelOneOrderById/'+orderId,
											method:'GET',
											header:{token:JSON.parse(that.token)},
											success(res) {
												console.log(res)
												if(res.data.message=='操作成功')
												{
													uni.showToast({
														title:'取消成功'
													}),
													that.update_order()
												}else{
													uni.showToast({
														title:res.data.message,
														icon:'error',
													})
												}
											},
											fail() {
												uni.showToast({
													title:'网络错误！',
													icon:'error',
												})
											}
										})
									}
								}
						})
					}
					else
					{
						uni.showModal({
							content:'确认删除此订单信息吗？',
							success(res){
								if(res.confirm)
								{
									uni.request({
										url:'http://101.34.159.132:8080/order/deleteOrderById/'+orderId,
										header:{token:JSON.parse(that.token)},
										method:'DELETE',
										success(res)
										{
											if(res.data.message=='操作成功'){
												uni.showToast({
													title:'删除成功'
												}),
												that.update_order()
											}else{
												uni.showToast({
													title:res.data.message,
													icon:'error',
												})
											}
										},
										fail() {
											uni.showToast({
												title:'删除失败',
												icon:'error'
											})
										}
									})
								}
	
							}
						})
					}
			
					
				}
				
			}
		}
	</script>

	<style>
		body{ background:#efefef}
		.order-item{
			height: auto;
			margin-top: 20rpx;
			background-color: #ffffff;
			border-radius: 20rpx;
		}
		.item-row{
			display: flex;
			flex-direction: row;
			justify-content: space-around;
		}
		.row-title{
			font-size: 40rpx;
			font-weight: 520;
			margin-top: 20rpx;
		}
		.row-sign-0{
			font-size: 35rpx;
			background-color:#00aa00;
			color: #ffffff;
			margin-top: 20rpx;
		}
		.row-sign-1{
			font-size: 35rpx;
			background-color:#ff0000;
			color: #ffffff;
			margin-top: 20rpx;
		}
		.item-column{
			display: flex;
			flex-direction: column;
			/* justify-content: space-around; */
			padding: 15rpx;
			margin-left: 15rpx;
			
			font-size: 30rpx;
			font-weight: 400;
		}
		.col-time{
			color: #9e9e9e;
			margin-bottom: 15rpx;
		}
		.col-cost{
			margin-bottom: 5rpx;
		}
		.col-wrapper{
			display: flex;
			align-items: center;
			justify-content: flex-end;
			border-top-style: solid;
			border-top-width: 1rpx;
			border-top-color: #e8e8e8;
			padding: 8rpx;
			margin-top: 20rpx;
			/* background-color: #18BC37; */
			
		}
		.button-0{
			margin-top: 5rpx;
			padding: 10rpx;
			border: 2rpx solid red;
			border-radius: 50rpx;
			color: #ff0000;
		}
		.button-1{
			margin-top: 5rpx;
			padding: 10rpx;
			border: 2rpx solid;
			border-color: #979797;
			border-radius: 50rpx;
			color: #979797;
		}
	</style>