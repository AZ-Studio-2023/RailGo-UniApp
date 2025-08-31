<template>
	<view class="ux-bg-grey5" style="height:100vh;">
		<view class="ux-bg-primary" style="height: 50rpx;">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">实时测速</text>
		</view>

		<view class="ux-padding ux-bg-grey5">
			
			<view class="speed-card">
				<view class="accuracy-btn" @click="toggleAccuracy">
					<view class="btn-circle" :class="{'is-high-accuracy': isHighAccuracy}">
						<text v-if="!isHighAccuracy" class="btn-text">低</text>
						<text v-if="isHighAccuracy" class="btn-text">高</text>
					</view>
				</view>

				<text class="speed-text">{{speed}}</text>
				<text class="unit-text">km/h</text>
				<view class="label-text">您的速度</view>
			</view>

			<view class="location-card">
				<view class="location-item">
					<text class="label-bold">经度</text>
					<text class="location-value">{{longitude}}</text>
				</view>
				<view class="location-item">
					<text class="label-bold">纬度</text>
					<text class="location-value">{{latitude}}</text>
				</view>
			</view>
			
			<view class="hemisphere-card">
				<view class="hemisphere-item">
					<text class="label-bold">半球</text>
					<text class="hemisphere-value">{{hemisphere_lat}} {{hemisphere_lon}}</text>
				</view>
			</view>
			
			<br>
			<view class="ux-padding-small ux-mb ux-h6 ux-text-center"
				style="background-color:#e9eef5;border:1px solid #114598;border-radius:10rpx;color:#114598;">
				<text class="ux-bold">定位服务由系统提供，测速信息仅供参考</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				"speed": 0,
				"longitude": 0,
				"latitude": 0,
				"hemisphere_lat": "未知",
				"hemisphere_lon": "未知",
				"isHighAccuracy": false 
			}
		},
		onLoad() {
			this.getLocationSpeed();
			this.startSpeedUpdate();
		},
		onUnload() {
			this.stopSpeedUpdate();
		},
		methods: {
			back: function() {
				uni.navigateBack()
			},
			toggleAccuracy() {
				this.isHighAccuracy = !this.isHighAccuracy;
				this.stopSpeedUpdate(); 
				this.startSpeedUpdate(); 
				uni.showToast({
					title: this.isHighAccuracy ? '已切换至高精度模式' : '已切换至普通精度模式',
					icon: 'none'
				});
			},
			startSpeedUpdate() {
				const interval = 200;
				this.updateInterval = setInterval(() => {
					this.getLocationSpeed();
				}, interval);
			},
			stopSpeedUpdate() {
				if (this.updateInterval) {
					clearInterval(this.updateInterval);
					this.updateInterval = null;
				}
			},
			getLocationSpeed() {
				uni.getLocation({
					type: 'wgs84',
					isHighAccuracy: this.isHighAccuracy, 
					success: (res) => {
						if (res.speed !== undefined && res.speed !== null) {
							this.speed = (res.speed * 3.6).toFixed(2);
						} else {
							this.speed = 0;
						}

						this.longitude = res.longitude.toFixed(6);
						this.latitude = res.latitude.toFixed(6);
					
						// 判断半球
						this.hemisphere_lat = res.latitude >= 0 ? '北半球' : '南半球';
						this.hemisphere_lon = res.longitude >= 0 ? '东半球' : '西半球';
					},
					fail: (err) => {
						console.error('获取地理位置失败:', err);
						this.speed = 0;
						this.longitude = 0;
						this.latitude = 0;
						this.hemisphere_lat = "未知";
						this.hemisphere_lon = "未知";
						uni.showToast({
							title: '获取速度和经纬度失败，请检查定位权限',
							icon: 'error'
						});
					}
				});
			}
		}
	}
</script>

<style>

	.speed-card {
		position: relative;
		background-color: #ffffff;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
		padding: 60rpx 40rpx;
		text-align: center;
		margin-top: 30rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.speed-text {
		font-size: 140rpx;
		font-weight: bold;
		color: #114598;
	}

	.unit-text {
		font-size: 40rpx;
		color: #666;
		margin-left: 10rpx;
		margin-top: -20rpx;
	}

	.label-text {
		font-size: 28rpx;
		color: #999;
		margin-top: 10rpx;
	}

	/* 高精度按钮容器 */
	.accuracy-btn {
		position: absolute;
		top: 20rpx;
		right: 20rpx;
		z-index: 10;
		cursor: pointer;
	}

	.btn-circle {
		width: 60rpx;
		height: 60rpx;
		border-radius: 50%;
		background-color: #114598; /* 普通精度默认蓝色 */
		display: flex;
		justify-content: center;
		align-items: center;
		color: #ffffff;
		font-size: 24rpx;
		transition: background-color 0.3s;
	}

	.btn-circle.is-high-accuracy {
		background-color: gold; 
	}

	.btn-text {
		font-weight: bold;
	}

	/* 经纬度信息卡片样式 */
	.location-card {
		background-color: #ffffff;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
		padding: 40rpx;
		margin-top: 30rpx;
		display: flex;
		justify-content: space-around;
	}

	.location-item {
		text-align: center;
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.label-bold {
		font-weight: bold;
		color: #333;
		font-size: 30rpx;
	}

	.location-value {
		font-size: 28rpx;
		color: #666;
		margin-top: 10rpx;
	}

	/* 半球信息卡片样式 */
	.hemisphere-card {
		background-color: #ffffff;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
		padding: 40rpx;
		margin-top: 30rpx;
		display: flex;
		justify-content: space-around;
	}

	.hemisphere-item {
		text-align: center;
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.hemisphere-value {
		font-size: 28rpx;
		color: #666;
		margin-top: 10rpx;
	}

	/* 提示信息卡片样式 */
	.info-box {
		background-color: #e6f7ff; /* 淡蓝色背景 */
		border: 1px solid #91d5ff; /* 边框颜色 */
		border-radius: 10rpx;
		padding: 20rpx;
		text-align: center;
		margin-top: 30rpx;
	}

	.info-text {
		color: #1890ff; /* 文字颜色 */
		font-size: 26rpx;
		font-weight: bold;
	}
</style>