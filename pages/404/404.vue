<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		
		<view class="ux-bg-primary" style="height: var(--status-bar-height);">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back" class="back-icon-area">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">Error</text>
		</view>
		
		<view class="ux-padding content-area">
			<view class="ux-text-center"><br><br>
				<text class="ux-h0 ux-text-primary">404</text>
				<br>
				<text class="ux-h4 ux-text-dark ux-bold">很抱歉，你来到了没有轨道的荒原...</text>
				
				<br><br><br>
				
				<view class="poem-container">
					<text class="ux-h5 ux-text-grey7">
						“众里寻他千百度，<br>
						蓦然回首，那人却在灯火阑珊处。”
					</text>
					<br><br>
					<text class="ux-text-small ux-text-grey6">— 辛弃疾《青玉案·元夕》</text>
				</view>
			</view>
			
			<!-- <br><br> -->
			<!-- <view class="video-placeholder"></view> -->
			<br>

			<button type="primary" style="background-color:#114598;color:#ffffff;" hover-class="ux-tap" 
				@click="back()">重回正轨</button>
			<br><br>
		</view>
		
<!-- 		<web-view v-if="randomVideoUrl" :src="randomVideoUrl" class="video-webview" :style="videoStyle"></web-view> -->
	</view>
</template>

<script>
	// 定义常量用于计算 web-view 尺寸
	const VIDEO_WIDTH = 90; // 90vw
	const ASPECT_RATIO = 9 / 16; // 16:9 比例
	
	export default {
		data() {
			return {
				videoUrls: [
					"//player.bilibili.com/player.html?isOutside=true&aid=112892259142606&bvid=BV1xNi3edEj7&cid=500001636041729&p=1",
					"//player.bilibili.com/player.html?isOutside=true&aid=525793456&bvid=BV1RM411s78B&cid=1050085824&p=1",
					"//player.bilibili.com/player.html?isOutside=true&aid=853706936&bvid=BV1vL4y1c7EL&cid=711536979&p=1"
				],
				randomVideoUrl: "",
				videoStyle: {}, 
			}
		},
		onLoad() {
			this.selectRandomVideo();
			setTimeout(() => {
				this.calculateVideoPosition();
			}, 300); 
		},
		onResize() {
			this.calculateVideoPosition();
		},
		methods: {
			selectRandomVideo() {
				const randomIndex = Math.floor(Math.random() * this.videoUrls.length);
				let selectedUrl = this.videoUrls[randomIndex];
				if (selectedUrl.startsWith("//")) {
					selectedUrl = "https:" + selectedUrl;
				}
				this.randomVideoUrl = selectedUrl;
			},
			calculateVideoPosition() {
				uni.createSelectorQuery().select('.video-placeholder').boundingClientRect(rect => {
					if (rect) {
						const screenWidth = uni.getSystemInfoSync().windowWidth;
						const rpxToPx = screenWidth / 750;

						const videoHeightPx = rect.width * ASPECT_RATIO; 
						
						this.videoStyle = {
							top: `${rect.top}px`, 
							left: `${rect.left}px`,
							width: `${rect.width}px`,
							height: `${videoHeightPx}px`,
						};
					}
				}).exec();
			},
			back: function() {
				if (getCurrentPages().length > 1) {
					uni.navigateBack();
				} else {
					uni.reLaunch({
						url: '/pages/index/index'
					});
				}
			}
		}
	}
</script>

<style>
	/* 404 数字的特大样式 */
	.ux-h0 {
		font-size: 200rpx; 
		font-weight: 900;
		line-height: 1.2;
	}
	
	/* 诗词容器样式 */
	.poem-container {
		margin: 20rpx auto; 
		padding: 20rpx 40rpx;
		border-left: 5rpx solid #114598; 
		width: max-content; 
		text-align: left; 
	}
	
	/* 返回图标区域，增加点击范围 */
	.back-icon-area { 
		width: 60rpx; 
		padding: 10rpx; 
	}
	
	/* 确保粗体生效 */
	.ux-bold {
		font-weight: bold;
	}

	/* 视频占位符（用于定位 web-view） */
	.video-placeholder {
		/* 占位符宽度，用于计算 web-view 的实际尺寸和位置 */
		width: 90vw; 
		max-width: 600rpx; 
		margin-left: auto;
		margin-right: auto;
		/* 使用 padding-bottom hack 来确保占位符的高度是 16:9 比例 */
		padding-bottom: calc(90vw * 0.5625); 
		max-height: 337.5rpx; /* 最大高度 */
		/* background-color: #e0e0e0; /* 调试用，可删除 */
	}

	/* web-view 组件：设置为 fixed 定位，并由 JS 赋予精确的 top/left/width/height */
	.video-webview {
		position: fixed;
		/* z-index 设高，虽然 web-view 默认就高，但保留以防万一 */
		z-index: 999; 
		/* 移除静态 width/height，它们将由 videoStyle 动态设置 */
	}
	
	/* 确保主体内容区域可以滚动 */
	.content-area {
		/* 确保内容区域能撑开，并且可以滚动 */
		padding-bottom: 20rpx;
	}
</style>