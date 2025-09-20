<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		<view class="ux-bg-primary" style="height: var(--status-bar-height);">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">数据库更新</text>
		</view>
		<view class="ux-padding">
			<view v-if="loading" class="card">
				<text class="card-title ux-text-center">{{ statusMessage }}</text><br><br>
				<view class="indeterminate-progress-container ux-progress-mt">
					<view class="indeterminate-progress-bar"></view>
				</view>
			</view>

			<view v-else class="card">
				<view class="ux-flex ux-align-items-center ux-space-between ux-pb-small">
					<view>
						<text class="card-title">{{ updateStatusText }}</text>
						<view class="card-subtitle">{{ hasUpdate ? '快去更新吧' : '已拥有最新数据' }}</view>
					</view>
					<text class="status-icon" :class="hasUpdate ? 'has-update-color' : 'no-update-color'">
						{{ hasUpdate ? '&#xe05e;' : '&#xe834;' }}
					</text>
				</view>
				
				<view class="line-divider"></view>
				
				<view class="version-row">
					<text class="label-bold">当前版本</text>
					<text class="version-value">{{ currentVersionText }}</text>
				</view>
				<view v-if="hasUpdate" class="version-row">
					<text class="label-bold">最新版本</text>
					<text class="version-value">{{ latestVersionText }}</text>
				</view>
			</view>
			
			<button v-if="hasUpdate" class="ux-button-primary ux-mt-large" @click="gotoDownload">立即更新</button>
			<button v-else class="ux-button-disabled ux-mt-large" disabled>已是最新</button>

			<br>
			<view class="ux-disclaimer ux-text-center">
				<text class="ux-bold ux-disclaimer-text">离线数据库信息可能滞后 请根据使用情况选择</text>
			</view>
		</view>
	</view>
</template>

<script>
	import uniGet from "@/scripts/req.js";
	
	export default {
		data() {
			return {
				loading: true,
				hasUpdate: false,
				currentVersion: "",
				currentVersionText: "",
				latestVersion: "",
				latestVersionText: "",
				statusMessage: "正在检查更新...",
				updateStatusText: "",
			}
		},
		async onShow() {
			this.checkUpdate();

		},
		methods: {
			back() {
				uni.navigateBack();
			},
			async checkUpdate() {
				this.loading = true;
				this.statusMessage = "正在检查更新...";
				
				// 获取本地版本号，并确保是数字类型
				const currentVersion = parseInt(uni.getStorageSync("offlineDataVersion")) || 0;
				this.currentVersionText = uni.getStorageSync("offlineDataVersionText") || "未知版本";
				
				try {
					// 请求服务器获取最新版本信息
					const finalInfoResponse = await uniGet("https://api.state.railgo.zenglingkun.cn/api/v1/info");
					
					if (finalInfoResponse.status === 200 && finalInfoResponse.data) {
						// 获取最新版本号，并确保是数字类型
						const latestVersion = parseInt(finalInfoResponse.data.latest_db);
						this.latestVersionText = finalInfoResponse.data.db;
						
						// 比较版本号
						if (latestVersion > currentVersion) {
							this.hasUpdate = true;
							this.updateStatusText = "发现新版本数据库";
						} else {
							this.hasUpdate = false;
							this.updateStatusText = "已是最新版本";
						}
					} else {
						throw new Error("获取版本信息失败");
					}
				} catch (e) {
					console.error("检查更新时发生错误:", e);
					this.statusMessage = "检查更新失败: " + (e.message || '未知错误');
					this.updateStatusText = "检查失败";
					this.hasUpdate = false;
				} finally {
					this.loading = false;
				}
			},
			gotoDownload() {
				uni.navigateTo({
					url: '/pages/oobe/download'
				});
			}
		}
	}
</script>

<style>
	/* 通用样式 */
	
	.ux-button-primary {
		background-color: #114598;
		color: #ffffff;
		font-size: 32rpx;
		margin-top: 60rpx;
	}
	.ux-button-disabled {
		background-color: #c0c0c0;
		color: #ffffff;
		font-size: 32rpx;
		border-radius: 50rpx;
		margin-top: 60rpx;
	}
	
	/* 卡片样式 */
	.card {
		background-color: #ffffff;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
		padding: 40rpx;
		margin-top: 30rpx;
	}
	
	.card-title {
		font-size: 40rpx;
		font-weight: bold;
		color: #114598;
	}
	
	.card-subtitle {
		font-size: 28rpx;
		color: #999;
		margin-top: 5rpx;
	}
	
	.line-divider {
		height: 1px;
		background-color: #eee;
		margin: 30rpx 0;
	}
	
	.version-row {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 20rpx;
	}
	
	.label-bold {
		font-weight: bold;
		color: #333;
		font-size: 30rpx;
	}
	
	.version-value {
		font-size: 28rpx;
		color: #666;
	}
	
	/* 状态图标样式 */
	.status-icon {
		font-family: 'Material Icons';
		font-size: 100rpx;
	}
	
	.has-update-color {
		color: #114598;
	}
	
	.no-update-color {
		color: #4cd964; /* 绿色 */
	}
	
	.ux-disclaimer {
		background-color:#e9eef5;
		border:1px solid #114598;
		border-radius:10rpx;
		color:#114598;
		padding: 20rpx;
	}
	
	/* 不确定进度条样式 */
	.indeterminate-progress-container {
		width: 100%;
		height: 6rpx;
		background-color: #ddd;
		border-radius: 3rpx;
		overflow: hidden;
	}
	
	.indeterminate-progress-bar {
		height: 100%;
		width: 30%;
		background-color: #114598;
		animation: scrollAnimation 1.5s linear infinite;
	}
	
	@keyframes scrollAnimation {
		0% {
			transform: translateX(-100%);
		}
		100% {
			transform: translateX(433%);
		}
	}
</style>