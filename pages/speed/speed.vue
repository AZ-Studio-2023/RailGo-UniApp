<template>
	<view class="ux-bg-grey5" style="height:100vh;">
		<view class="ux-bg-primary" style="height: var(--status-bar-height);">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">实时测速</text>
		</view>

		<view class="ux-padding ux-bg-grey5">
			<view class="speed-card">
				<view class="mode-btn-container">
					<view 
						class="mode-btn" 
						:class="{'active': currentMode === 'high'}" 
						@click="changeMode('high')"
					>
						高精度
					</view>
					<view 
						class="mode-btn" 
						:class="{'active': currentMode === 'low'}" 
						@click="changeMode('low')"
					>
						普通
					</view>
					<view 
						class="mode-btn" 
						:class="{'active': currentMode === 'sensor'}" 
						@click="changeMode('sensor')"
					>
						非定位
					</view>
				</view>

				<text class="speed-text">{{speed}}</text>
				<text class="unit-text">km/h</text>
				<view class="label-text">您的速度</view>
			</view>

			<view class="sensor-info-card" v-if="currentMode === 'sensor'">
				<view class="sensor-item">
					<text class="label-bold">加速度</text>
					<text class="sensor-value">{{acceleration.toFixed(2)}}</text>
					<text class="unit-small">m/s²</text>
				</view>
			</view>

			<view class="location-card" v-if="currentMode !== 'sensor'">
				<view class="location-item">
					<text class="label-bold">经度</text>
					<text class="location-value">{{longitude}}</text>
				</view>
				<view class="location-item">
					<text class="label-bold">纬度</text>
					<text class="location-value">{{latitude}}</text>
				</view>
			</view>
			
			<view class="hemisphere-card" v-if="currentMode !== 'sensor'">
				<view class="hemisphere-item">
					<text class="label-bold">半球</text>
					<text class="hemisphere-value">{{hemisphere_lat}} {{hemisphere_lon}}</text>
				</view>
			</view>
			
			<br>
			<view class="ux-padding-small ux-mb ux-h6 ux-text-center"
				style="background-color:#e9eef5;border:1px solid #114598;border-radius:10rpx;color:#114598;">
				<text class="ux-bold">非定位模式速度仅为估算，不提供位置信息</text>
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
				"currentMode": "low", // 'high', 'low', 'sensor'
				"updateInterval": null,
				"acceleration": 0,
				"lastSpeedUpdateTime": Date.now(),
			}
		},
		onLoad() {
			this.startSpeedUpdate();
		},
		onUnload() {
			this.stopSpeedUpdate();
		},
		methods: {
			back: function() {
				uni.navigateBack()
			},
			changeMode(mode) {
				if (this.currentMode === mode) return;
				this.currentMode = mode;
				this.speed = 0; // Reset speed on mode change
				this.stopSpeedUpdate();
				this.startSpeedUpdate();
				uni.showToast({
					title: `已切换至${mode === 'high' ? '高精度' : (mode === 'low' ? '普通' : '非定位')}模式`,
					icon: 'none'
				});
			},
			startSpeedUpdate() {
				// Clear any existing intervals first
				this.stopSpeedUpdate();

				// Use a single, unified update interval
				const interval = 1000;
				this.updateInterval = setInterval(() => {
					if (this.currentMode === 'sensor') {
						this.getSensorSpeed();
					} else {
						this.getLocationSpeed();
					}
				}, interval);
			},
			stopSpeedUpdate() {
				if (this.updateInterval) {
					clearInterval(this.updateInterval);
					this.updateInterval = null;
				}
				// Stop listening to sensors when not in use
				try {
					uni.stopAccelerometer();
				} catch (e) {
					console.error("Failed to stop accelerometer", e);
				}
			},
			getLocationSpeed() {
				uni.getLocation({
					type: 'wgs84',
					isHighAccuracy: this.currentMode === 'high',
					success: (res) => {
						const currentSpeed = (res.speed * 3.6);
						if (currentSpeed >= 0) {
							this.speed = currentSpeed.toFixed(2);
						}
						this.longitude = res.longitude.toFixed(6);
						this.latitude = res.latitude.toFixed(6);
					
						this.hemisphere_lat = res.latitude >= 0 ? '北半球' : '南半球';
						this.hemisphere_lon = res.longitude >= 0 ? '东半球' : '西半球';
					},
					fail: (err) => {
						console.error('获取地理位置失败:', err);
						uni.showToast({
							title: '请开启定位权限',
							icon: 'error'
						});
					}
				});
			},
			getSensorSpeed() {
				const g = 9.80665; // Gravity
				let lastSpeed = this.speed / 3.6; // Convert back to m/s for calculation
				let lastTime = this.lastSpeedUpdateTime;
				let lastAcceleration = { x: 0, y: 0, z: 0 };

				uni.onAccelerometerChange((res) => {
					const now = Date.now();
					const deltaTime = (now - lastTime) / 1000; // Time in seconds
					lastTime = now;

					// Calculate acceleration magnitude
					const accelerationMagnitude = Math.sqrt(
						Math.pow(res.x, 2) + 
						Math.pow(res.y, 2) + 
						Math.pow(res.z, 2)
					);

					// A simple filter to remove gravity and noise.
					// This is a very basic method and can be improved with more complex filters.
					const horizontalAcceleration = Math.abs(accelerationMagnitude - g);
					this.acceleration = horizontalAcceleration;

					// Integrate acceleration to get change in velocity
					const deltaSpeed = horizontalAcceleration * deltaTime;
					lastSpeed += deltaSpeed;

					// Update speed, converting back to km/h
					this.speed = (lastSpeed * 3.6).toFixed(2);
					this.lastSpeedUpdateTime = now;
				});

				uni.startAccelerometer({
					success() {
						console.log('Accelerometer started.');
					},
					fail(e) {
						console.error("Failed to start accelerometer", e);
						uni.showToast({
							title: '设备不支持加速度计',
							icon: 'error'
						});
						this.changeMode('low'); // Fallback to a GPS mode
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

	.mode-btn-container {
		position: absolute;
		top: 20rpx;
		right: 20rpx;
		display: flex;
		gap: 10rpx;
	}
	
	.mode-btn {
		padding: 10rpx 20rpx;
		border-radius: 30rpx;
		background-color: #e9eef5;
		color: #114598;
		font-size: 24rpx;
		font-weight: bold;
		cursor: pointer;
		transition: background-color 0.3s, color 0.3s;
	}
	
	.mode-btn.active {
		background-color: #114598;
		color: #ffffff;
	}

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

	.sensor-info-card {
		background-color: #ffffff;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
		padding: 40rpx;
		margin-top: 30rpx;
		display: flex;
		justify-content: center;
	}

	.sensor-item {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.sensor-value {
		font-size: 40rpx;
		font-weight: bold;
		color: #114598;
	}

	.unit-small {
		font-size: 24rpx;
		color: #666;
		margin-top: 5rpx;
	}
</style>