<!-- 点击定位跳转到用户当前位置，点击停车将会根据地图中心点来就近获取停车场，点击列表将定位到停车场（这时候再点击停车会得到当前停车场距离为0，看似睿智实际影响不大，因为目的地必须离停车场较近） -->
<template>
	<view style="display: flex; flex-direction: column;">
		<view style="display: flex; flex-direction: row;">
			<image src='../../static/R-C.jpeg' class="avatar" @click="toInfo"></image>
			<!-- <text >{{username}}</text> -->
			<button class="button" @click="update_location" :loading="btn1">当前定位</button>
			<button class="button" @click="update_park2" :loading="btn2">查找附近</button>
		</view>
		
		<view class="datetime-line" style="">
			<view class="datetime-ceil">
				<picker class="date" mode="date" :value="date1" :start="date1_start" :end="date1_end"
					@change="ChangeDate1">
					<view class="uni-input">{{date1}}</view>
				</picker>
				
				<picker class="time" mode="time" :value="time1" 
					@change="ChangeTime1">
					<view class="uni-input">{{time1}}</view>
				</picker>
			</view>
			
			<view style=""><text>至</text></view>
			
			<view class="datetime-ceil">
				<picker class="date" mode="date" :value="date2" :start="date2_start" :end="date2_end"
					@change="ChangeDate2">
					<view class="uni-input">{{date2}}</view>
				</picker>
				<picker class="time" mode="time" :value="time2" 
					@change="ChangeTime2">
					<view class="uni-input">{{time2}}</view>
				</picker>
			</view>
		</view>
		
		<map id="Mymap"
			style="width: 750rpx; height: 750rpx;"
			scale="16" 
			
			show-compass="true"
			enable-rotate="true"
			enable-overlooking="true"
			enable-traffic="true"
			enable-poi="true"
			enable-building="true"
			show-location="true"
			
			:markers="parklot" 
			:circles="circle"
			
			@markertap="markertap">
			
		</map>
		
		<scroll-view scroll-y="true" style="height: 461rpx;">
			<view class="cell" v-for="(item,index) in parklot" @click="handle(item)" :key="index" >
				<text>
					(id:{{item.id}}){{item.description}}
					{{item.category}}
					剩余车位：{{item.space_count}}<!-- /{{item.total_space}} -->辆
					距我：{{getDistance(item.distance)}}
				</text>
			</view>
		</scroll-view>
		
	</view>
</template>

<script>
	function initDate(type) {//返回格式化Date
		let tmp = 0;
		if(typeof(type)=='number')
			tmp += 24*60*60*1000*type
		
		const date=new Date(new Date().getTime()+tmp)
		let year = date.getFullYear();
		let month = date.getMonth() + 1;
		let day = date.getDate();
	
		if (type === 'end') {//最多提前一年预约
			year = year + 1;
		}
		month = month > 9 ? month : '0' + month;;
		day = day > 9 ? day : '0' + day;
	
		return `${year}-${month}-${day}`;
	};
	function initTime() {//返回格式化Date
		const date = new Date();
		let hour = date.getHours();
		let minute = date.getMinutes()

		hour = hour > 9 ? hour : '0' + hour;
		minute = minute > 9 ? minute : '0' + minute;
	
		return `${hour}:${minute}`;
	};
	export default {
		data() {
			return {
				btn1:false,
				btn2:false,
				
				date1:initDate(1),
				time1:initTime(),
				date2:initDate(2),
				time2:initTime(),
				date1_start:initDate(),
				date1_end:initDate('end'),
				date2_start:initDate(),
				date2_end:initDate('end'),
				
				token:'',
				parkingLot:[],//停车场全信息
				parklot:[],//停车场展示信息（按距离排序）     id等价于该停车场在parkingLot中的下标
				circle:[],//这是一个圆
				begTime:'',//时间段
				endTime:'',
				username:''
			}
		},
		onReady() {
			uni.setNavigationBarTitle({
			    title: '预约车位'
			});
		},
		onShow() {//定位+定时(不计入时间段)
			var that=this;
			uni.getStorage({
				key:'token',
				success(res) {
					that.token=res.data;
					//console.log(that.token);
					//直接更新时间至明天到后天
					that.update_datetime();
					that.update_location();
				}
			})
			uni.getStorage({
				key:'userId',
				success(res) {
					that.userId=res.data;
					//console.log(that.userId);
				},
			})
			uni.getStorage({
				key:'username',
				success(res){
					that.username=JSON.parse(res.data)
				}
			})
		},
		methods: {
			getDistance(dis){
				if(dis>=1)
					return dis+'km'
				else
					return (dis*1000)+'m'
			},
			handle(item) {//跳转到停车场位置
				console.log(item.id);
				var idx=item.id;
				var that=this;
				// if(that.parkingLot[idx].space_count==0){
				// 	uni.showToast({
				// 		title:'无空位',
				// 		icon:'error'
				// 	})
				// 	return;
				// }
				
				var X=that.parkingLot[idx].latitude;
				var Y=that.parkingLot[idx].longitude;
				
				that._map=uni.createMapContext("Mymap",that);
				that._map.moveToLocation({
					latitude:X,
					longitude:Y,
					success(r) {
						// console.log('跳转到：'+X+', '+Y);
						// that._map.getCenterLocation({
						// 	success(r2) {
						// 		var X2=r2.latitude;
						// 		var Y2=r2.longitude;
						// 		console.log('跳转位置成功：'+X2+','+Y2);
						// 	}
						// })
					},
					fail(){
						console.log('定位位置失败');
						uni.showToast({
							title:'请重试',
							icon:'loading',
						})
					},
				})
			},
			update_datetime(){//更新时间段
				this.begTime = new Date(this.date1+" "+this.time1+":00");
				this.endTime = new Date(this.date2+" "+this.time2+":00");
				// console.log(this.begTime);
				// console.log(this.endTime);
				this.update_park2()
			},
			ChangeDate1(e){
				this.date1=e.detail.value;
				this.update_datetime();
			},
			ChangeDate2(e){
				this.date2=e.detail.value;
				this.update_datetime();
			},
			ChangeTime1(e){
				this.time1=e.detail.value;
				this.update_datetime();
			},
			ChangeTime2(e){
				this.time2=e.detail.value;
				this.update_datetime();
			},
			
			markertap(e){//点击停车场图标弹出子窗口
				var that=this;
				var idx=e.detail.markerId;
				
				if(that.parkingLot[idx].space_count==0){
					uni.showToast({
						title:'无空位',
						icon:'error'
					})
					return
				}
				uni.$emit('page-popup', {
					// title: '(id:'+idx+")"+that.parkingLot[idx].description,
					title: that.parkingLot[idx].description,
					content: '距我：'+that.parkingLot[idx].distance+'km',
					begTime: that.begTime,
					endTime: that.endTime,
					parking_lot_id: that.parkingLot[idx].parking_lot_id,
					token: that.token,
					userId: that.userId,
				});
				const subNVue = uni.getSubNVueById('sub_park')
				subNVue.show('slide-in-top', 250,()=>{
					console.log('子窗体打开成功');
				})
			},
			
			Rad(d) {
				return d * Math.PI / 180.0; //经纬度转换成三角函数中度分表形式。
			},
			getMapDistance(lat1, lng1, lat2, lng2) {
				/*
				 计算距离，参数分别为第一点的纬度，经度；第二点的纬度，经度
				 默认单位km
				*/
				var radLat1 = this.Rad(lat1);
				var radLat2 = this.Rad(lat2);
				var a = radLat1 - radLat2;
				var b = this.Rad(lng1) - this.Rad(lng2);
				var s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a / 2), 2) +
					Math.cos(radLat1) * Math.cos(radLat2) * Math.pow(Math.sin(b / 2), 2)));
				s = s * 6378.137; // EARTH_RADIUS;
				s = Math.round(s * 10000) / 10000; //输出为公里
				//s=s.toFixed(2);
				return s;
			},
			update_location(){//定位到当前位置
				var that=this;
				that.btn1=true;
				uni.getLocation({
					type: 'gcj02',
					altitude: true,
					success(res) {
						var X=res.latitude;
						var Y=res.longitude;
						console.log('更新位置: '+X+'，'+Y);
						
						that._map=uni.createMapContext("Mymap",that);
						that._map.moveToLocation({
							latitude:X,
							longitude:Y,
							success(r) {
								// that._map.getCenterLocation({
								// 	success(r2) {
								// 		var X2=r2.latitude;
								// 		var Y2=r2.longitude;
								// 		console.log('更新位置成功：'+X2+','+Y2);
								// 	}
								// })
							},
							fail(){
								console.log('定位位置失败');
								uni.showToast({
									title:'定位位置失败',
									icon:'error',
								})
							},
							complete(){
								console.log('更新停车场');
								setTimeout(()=>{that.update_park2();}, 500);
							}
						})
						// that.control[0]={
						// 					"position":{
						// 									"left":0,
						// 									"top":0,
						// 								},
						// 					"iconPath":'/static/point.png',
						// 				};
					},
					fail() {
						console.log('获取位置失败');
						uni.showToast({
							title:'获取位置失败',
							icon:'error',
						})
					},complete() {
						that.btn1=false
					}
				})
				
			},
			update_park(){//返回停车场
				var that=this;
				that._map=uni.createMapContext("Mymap",that);
				that._map.getCenterLocation({
					success(r) {
						//console.log(r);
						var X=r.latitude;
						var Y=r.longitude;
						// that.circle=[];
						// that.circle[0]={
						// 					"latitude":X,
						// 					"longitude":Y,
						// 					"radius":300,
						// 					"color":'#00aa00',
						// 				};
						uni.request({
							url:'http://101.34.159.132:8080/parkingLot/findAllParkingLot',
							method:'GET',
							header:{token:JSON.parse(that.token)},//无语
							success(res) {
								console.log(res);
								if(res.data.code!="200"){
									console.log("停车场信息获取失败！");
									return;
								}
								//console.log("停车场信息获取成功！");
								that.parkingLot=res.data.data;
								//console.log(that.parkingLot);
								//that.description=res.data.data[0].description
								that.parklot=[];
								for(var i=0;i<that.parkingLot.length;++i)
								{
									let p={}
									let key='latitude'
									let value=that.parkingLot[i].latitude
									p[key] = value
									key = 'longitude'
									value = that.parkingLot[i].longitude
									p[key]=value
									key='description'
									value=that.parkingLot[i].description
									p[key]=value
									key='callout'
									value={
												content: 'id:'+i,
												borderRadius: 5,
												display: "ALWAYS",
												padding: 7,
												bgColor: "#FFFFFF",
											}
									p[key]=value
									key='distance'
									value=that.getMapDistance(
										X,Y,that.parkingLot[i].latitude,that.parkingLot[i].longitude)
									p[key]=value
									
									that.parkingLot[i][key]=value
									
									key = 'iconPath'
									value = '/static/point.png'
									p[key]=value
									key='id'
									value=i
									p[key]=value
									
									that.parklot.push(p)
									//console.log(p)
								}
								that.parklot.sort(
								(a,b)=>{
									return (a.distance<b.distance) ? -1 : (a.distance>b.distance) ? 1 :0
								}
								)
							},
							fail() {
								console.log('fail')
							}
						})
					},
					fail(){
						console.log('定位失败');
					},
				})
				
			},
			update_park2(){//根据时间段返回停车场
				var that=this;
				that.btn2=true;
				that.parklot=[];
				that.parkingLot=[];
				if(!that.begTime||!that.endTime){
					uni.showToast({
						title:'请选择停车时段',
						icon:'error',
					})
					that.btn2=false;
					return;
				}
				if(that.begTime>=that.endTime){
					uni.showToast({
						title:'停车时段非法',
						icon:'error',
					})
					that.btn2=false;
					return;
				}
				
				// console.log(that.begTime.toISOString());
				// console.log(that.endTime.toISOString());
				
				var cur_time = new Date().getTime();
				if(that.begTime.getTime()-cur_time<30*60*1000){
					uni.showToast({
						title:'提前半小时预约',
						icon:'error',
					})
					that.btn2=false;
					return;
				}
				
				that._map=uni.createMapContext("Mymap",that);
				that._map.getCenterLocation({
					success(r) {
						var X=r.latitude;
						var Y=r.longitude;
						console.log('查找附近：'+X+','+Y);
						
						// that.circle=[];
						// that.circle[0]={
						// 					"latitude":X,
						// 					"longitude":Y,
						// 					"radius":300,
						// 					"color":'#00aa00',
						// 				};
						
						// var d={'beginTime':that.begTime,'endTime':that.endTime};
						uni.request({
							url:'http://101.34.159.132:8080/parkingLot/checkAllAvailableParkingLotDuringPeriod',
							method:'POST',
							header:{token:JSON.parse(that.token)},//无语
							data:{'beginTime':that.begTime.toISOString(),'endTime':that.endTime.toISOString()},
							
							success(res) {
								console.log(res);
								if(res.data.code!="200"){
									console.log("停车场信息获取失败！");
									uni.showToast({
										title:'请求失败',
										icon:'error',
									})
									that.btn2=false;
									return;
								}
								//console.log("停车场信息获取成功！");
								that.parkingLot=res.data.data;
								//console.log(that.parkingLot);
								for(var i=0;i<that.parkingLot.length;++i)
								{
									var idx=i
									let p={}
									let key='latitude'
									let value=that.parkingLot[idx].latitude
									p[key] = value
									key = 'longitude'
									value = that.parkingLot[idx].longitude
									p[key]=value
									
									key='space_count'
									value=that.parkingLot[idx].space_count
									p[key]=value
									// p['total_space']=value
									
									key='category'
									value=that.parkingLot[idx].category
									p[key]=value
									
									key='description'
									value=that.parkingLot[idx].description
									p[key]=value
									
									key='callout'
									value={
												content: that.parkingLot[idx].description,
												borderRadius: 5,
												display: "ALWAYS",
												padding: 7,
												bgColor: "#FFFFFF",
											}
									p[key]=value
									
									key='distance'
									value=that.getMapDistance(
										X,Y,that.parkingLot[idx].latitude,that.parkingLot[idx].longitude)
									p[key]=value
									
									that.parkingLot[idx][key]=value//停车场扩展信息
									
									key = 'iconPath'
									value = '/static/parking.png'
									p[key]=value
									
									key='id'
									value=idx
									p[key]=value
									
									that.parklot[idx]=p
								}
								that.parklot.sort(   //有空位且距离近优先
								(a,b)=>{
									if(a.space_count==0){
										if(b.space_count==0) return (a.distance<b.distance) ? -1 : (a.distance>b.distance) ? 1 :0;
										else return 1;
									}
									else if(b.space_count==0) return -1;
									else return (a.distance<b.distance) ? -1 : (a.distance>b.distance) ? 1 :0;
								}
								)
							},
							fail() {
								console.log('fail')
								uni.showToast({
									title:'网络错误',
									icon:'error',
								})
							},
							complete() {
								that.btn2=false;
							}
						})
					},
					fail(){
						console.log('定位失败');
						uni.showToast({
							title:'请重试',
							icon:'loading',
						})
					},
					complete() {
						that.btn2=false;
					}
				})
				
			},
			toInfo(){
				uni.switchTab({
					url:'/pages/User/User',
				})
			}
		},
		
	}
		
	
</script>

<style>
	.coverview {
	 	    position: absolute;
	 	    left: 0;
	 	    right: 0;
	 	    top: 0rpx;
	 	    height: 150rpx;
	 	    border-width: 10rpx;
	 	    border-color: #4CD964;
	 	}
	 	
	 	.example-body {
	 		background-color: #fff;
	 		padding: 10px;
	 	}
	 	
	 	.datetime-ceil{
	 		display: flex; 
	 		flex-direction: row;
	 		/* flex: 2; */
	 		justify-content: space-between;
			
			padding: 10rpx;
			border: 2rpx solid;
			border-color: #979797;
			border-radius: 20rpx;
	 	}
	 	
	 	.datetime-line{
	 		display: flex; 
	 		flex-direction: row;
	 		justify-content: space-around;
	 		padding: 10rpx;
			
			border-top-style: dotted;
			border-top-width: 1rpx;
			border-top-color: #e8e8e8;
	 	}
	 	
	 	.cell {
	 		margin: 10rpx;
	 		padding: 20rpx 0;
	 		top: 10rpx;
	 		align-items: center;
	 		justify-content: center;
	 		border-radius: 10rpx;
	 		background-color: #ebebeb;
	 	}
		.avatar{
			padding: 25rpx;
			width: 50rpx;
			height: 50rpx;
		}
		.button{
			padding: 0rpx;
			width: 250rpx;
			height: 90rpx;
			font-size: medium;
			margin-top: 8rpx;
		}
		.time{
			margin-left: 20rpx;
			text-align: right;
		}
		.date{
			
		}
</style>