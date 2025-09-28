<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		<view class="ux-bg-primary" style="height: var(--status-bar-height);">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">更新管理</text>
		</view>
		<view class="ux-padding">
			<uni-section style="background-color:#eeeeee;margin-left:-1.2vh;" type="line" title="软件本体" title-font-size="34rpx"></uni-section>
			
			<view v-if="loading" class="card">
				<text class="card-title ux-text-center">{{ statusMessage }}</text><br><br>
				<view class="indeterminate-progress-container ux-progress-mt">
					<view class="indeterminate-progress-bar"></view>
				</view>
			</view>
			
			<view v-else class="card">
				<view class="ux-flex ux-align-items-center ux-space-between ux-pb-small">
					<view>
						<text class="card-title">{{ appUpdateStatusText }}</text>
						<view v-if="hasAppUpdate" class="card-subtitle">快去更新吧</view>
						<view v-else-if="isBeta" class="card-subtitle">您正在使用测试版软件</view>
						<view v-else class="card-subtitle">已是最新</view>
					</view>
					<text class="status-icon" :class="hasAppUpdate ? 'has-update-color' : 'no-update-color'">
						{{ hasAppUpdate ? '&#xe05e;' : '&#xe834;' }}
					</text>
				</view>
				
				<view class="line-divider"></view>
				
				<view class="version-row">
					<text class="label-bold">当前版本</text>
					<text class="version-value">{{ currentAppVersionText }}</text>
				</view>
				<view v-if="hasAppUpdate" class="version-row">
					<text class="label-bold">最新版本</text>
					<text class="version-value">{{ latestAppVersionText }}</text>
				</view>
			</view>
			
			<button v-if="hasAppUpdate" class="ux-button-primary ux-mt-large" @click="gotoDownloadApp">立即更新软件</button>
			<button v-else-if="isBeta" class="ux-button-disabled ux-mt-large" disabled>测试版本</button>
			<button v-else class="ux-button-disabled ux-mt-large" disabled>软件已是最新</button>
			<br>				<uv-divider text="·" :hairline="false"></uv-divider>

			<uni-section style="background-color:#eeeeee;margin-left:-1.2vh;" type="line" title="数据库" title-font-size="34rpx"></uni-section>
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
						<view class="card-subtitle">{{ hasDbUpdate ? '快去更新吧' : '已拥有最新数据' }}</view>
					</view>
					<text class="status-icon" :class="hasDbUpdate ? 'has-update-color' : 'no-update-color'">
						{{ hasDbUpdate ? '&#xe05e;' : '&#xe834;' }}
					</text>
				</view>
				
				<view class="line-divider"></view>
				
				<view class="version-row">
					<text class="label-bold">当前版本</text>
					<text class="version-value">{{ currentDbVersionText }}</text>
				</view>
				<view v-if="hasDbUpdate" class="version-row">
					<text class="label-bold">最新版本</text>
					<text class="version-value">{{ latestDbVersionText }}</text>
				</view>
			</view>
			
			<button v-if="hasDbUpdate" class="ux-button-primary ux-mt-large" @click="gotoDownload">立即更新数据库</button>
			<button v-else class="ux-button-disabled ux-mt-large" disabled>数据库已是最新</button>

		</view>
	</view>
</template>

<script>
import {uniGet} from "@/scripts/req.js";
	
	export default {
		data() {
			return {
				loading: true,
				statusMessage: "正在检查更新...",

				// 数据库更新状态
				hasDbUpdate: false,
				currentDbVersion: 0,
				currentDbVersionText: "",
				latestDbVersion: 0,
				latestDbVersionText: "",
				updateStatusText: "",

				// 软件更新状态
				hasAppUpdate: false,
				isBeta: false,
				currentAppVersion: 0,
				currentAppVersionText: "",
				latestAppVersion: 0,
				latestAppVersionText: "",
				appUpdateStatusText: "",
				
			}
		},
		async onShow() {
			this.checkAllUpdates();
		},
		methods: {
			back() {
				uni.navigateBack();
			},
			async checkAllUpdates() {
				this.loading = true;
				this.statusMessage = "正在检查更新...";
				
				// 获取本地数据库版本号
				this.currentDbVersion = parseInt(uni.getStorageSync("offlineDataVersion")) || 0;
				this.currentDbVersionText = uni.getStorageSync("offlineDataVersionText") || "未知版本";
				
				// 获取本地软件版本号
				this.currentAppVersion = parseInt(uni.getStorageSync("version")) || 0;
				this.currentAppVersionText = uni.getStorageSync("versionText") || "未知版本";

				try {
					// 请求服务器获取最新版本信息
					const finalInfoResponse = await uniGet("https://api.state.railgo.zenglingkun.cn/api/v1/info");
					
					if (finalInfoResponse.status === 200 && finalInfoResponse.data) {
						const data = finalInfoResponse.data;

						// --- 数据库更新逻辑 ---
						this.latestDbVersion = parseInt(data.latest_db);
						this.latestDbVersionText = data.db;
						if (this.latestDbVersion > this.currentDbVersion) {
							this.hasDbUpdate = true;
							this.updateStatusText = "发现新版本数据库";
						} else {
							this.hasDbUpdate = false;
							this.updateStatusText = "已是最新版本";
						}
						
						// --- 软件更新逻辑 ---
						this.latestAppVersion = parseInt(data.latest_pack);
						this.latestAppVersionText = data.pack;
						
						if (this.latestAppVersion > this.currentAppVersion) {
							this.hasAppUpdate = true;
							this.isBeta = false;
							this.appUpdateStatusText = "发现新版本软件";
						} else if (this.latestAppVersion < this.currentAppVersion) {
							this.hasAppUpdate = false;
							this.isBeta = true;
							this.appUpdateStatusText = "哇塞！";
						} else {
							this.hasAppUpdate = false;
							this.isBeta = false;
							this.appUpdateStatusText = "已是最新版本";
						}

					} else {
						throw new Error("获取版本信息失败");
					}
				} catch (e) {
					console.error("检查更新时发生错误:", e);
					this.statusMessage = "检查更新失败: " + (e.message || '未知错误');
					this.updateStatusText = "检查失败";
					this.appUpdateStatusText = "检查失败";
					this.hasDbUpdate = false;
					this.hasAppUpdate = false;
				} finally {
					this.loading = false;
				}
			},
			gotoDownload() {
				uni.navigateTo({
					url: '/pages/oobe/download'
				});
			},
			async gotoDownloadApp() {
				if (typeof plus === 'undefined') {
					uni.showModal({
						title: '提示',
						content: '此功能仅支持原生APP环境',
						showCancel: false
					});
					return;
				}

				uni.showLoading({
					title: '正在获取下载地址...',
					mask: true
				});
				
				let downloadUrl = '';
				try {
					const downloadResponse = await uniGet("https://api.state.railgo.zenglingkun.cn/api/v1/url/pack/android");
					if (downloadResponse.status === 200 && downloadResponse.data && downloadResponse.data.url) {
						downloadUrl = downloadResponse.data.url;
					} else {
						throw new Error("下载地址获取失败");
					}
				} catch (e) {
					uni.hideLoading();
					uni.showModal({
						title: '获取失败',
						content: `无法获取下载地址，请稍后再试。错误信息：${e.message}`,
						showCancel: false
					});
					return;
				}
				
				uni.showLoading({
					title: '正在下载 0%',
					mask: true
				});

				const downloadTask = uni.downloadFile({
					url: downloadUrl,
					success: (res) => {
						if (res.statusCode === 200) {
							uni.hideLoading();
							uni.showToast({
								title: '下载成功，正在安装...',
								icon: 'none',
								duration: 2000
							});
							// 调用plus.runtime.install安装应用
							plus.runtime.install(res.tempFilePath, {
								force: false
							}, (res) => {
								uni.showToast({
									title: '安装成功',
									icon: 'success'
								});
							}, (err) => {
								uni.showModal({
									title: '安装失败',
									content: `安装文件可能已损坏，或权限不足。错误码：${err.code}，错误信息：${err.message}`,
									showCancel: false
								});
								console.error('安装失败:', err);
							});
						} else {
							uni.hideLoading();
							uni.showModal({
								title: '下载失败',
								content: '下载更新包失败，请稍后再试。',
								showCancel: false
							});
						}
					},
					fail: (err) => {
						uni.hideLoading();
						uni.showModal({
							title: '下载失败',
							content: `网络错误，无法下载。错误码：${err.errMsg}`,
							showCancel: false
						});
					}
				});

				// 监听下载进度
				downloadTask.onProgressUpdate((res) => {
					uni.showLoading({
						title: `正在下载 ${res.progress}%`,
						mask: true
					});
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