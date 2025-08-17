<template>
	<view class="ux-bg-grey5 page">
		<view class="ux-bg-primary status-bar"></view>
		<view class="ux-padding">
			<view class="ux-flex ux-space-between ux-align-items-center">
				<view>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</view>
				<image class="ux-mb-small" src="/static/index-logo.png" mode="widthFix" style="width:250rpx;"></image>
				<view hover-class="ux-tap">
					<navigator class="ux-border-radius" url="/pages/about/about">
						<text class="icon section-icon ux-pt-small">&#xe5d4;</text>
					</navigator>
				</view>
			</view>
			<br>
			<text class="ux-pl-small ux-opacity-6 ux-text-small">{{this.title}}</text>
			<view class="ux-border-radius-large notice">
				<view class="left">
					<text class="icon ux-color-primary">&#xe0b9;</text>
					<text class="text">&nbsp;公告</text>
				</view>
				<view class="center">
					<swiper vertical autoplay interval="2500" duration="300" circular>
						<swiper-item v-for="(item, index) in items" :key="index"
							class="ux-pl-small ux-opacity-8 ux-text-small">
							{{ item }}
						</swiper-item>
					</swiper>
				</view>
			</view>

			<view class="ux-mt-small ux-flex1">
				<view class="ux-flex ux-rows ux-wrap ux-space-between">
					<navigator class="ux-th ux-bg-white ux-border-radius-large ux-padding ux-mr-small"
						style="flex:auto;width:1rpx;" hover-class="ux-tap" url="/pages/train/query">
						<text class="icon section-icon ux-color-purple">&#xe192;</text>
						<br>
						<text class="ux-text">车次</text>
						<br>
						<text class="ux-text-small ux-opacity-8">查询时刻、开行日等信息。</text>
						<br>
						<br>
						<view class="ux-text-right ux-mr-small">
							<text class="icon">&#xe5c8;</text>
						</view>
					</navigator>
					<navigator class="ux-th ux-bg-white ux-border-radius-large ux-padding ux-ml-small"
						style="flex:auto;width:1rpx;" hover-class="ux-tap" url="/pages/station/query">
						<text class="icon section-icon ux-color-cyan1">&#xe88a;</text>
						<br>
						<text class="ux-text">车站</text>
						<br>
						<text class="ux-text-small ux-opacity-8">查询通过车次、线路等信息。</text>
						<br>
						<br>
						<view class="ux-text-right ux-mr-small">
							<text class="icon">&#xe5c8;</text>
						</view>
					</navigator>
				</view>
				<br>
				<view class="ux-flex ux-rows ux-wrap ux-space-between">
					<navigator class="ux-th ux-bg-white ux-border-radius-large ux-padding ux-mr-small"
						style="flex:auto;width:1rpx;" hover-class="ux-tap" url="/pages/emu/query">
						<text class="icon section-icon ux-color-orange1">&#xe570;</text>
						<br>
						<text class="ux-text">动车组</text>
						<br>
						<text class="ux-text-small ux-opacity-8">查询动车组配属和运行交路。</text>
						<br>
						<br>
						<view class="ux-text-right ux-mr-small">
							<text class="icon">&#xe5c8;</text>
						</view>
					</navigator>
					<view class="ux-th ux-bg-white ux-border-radius-large ux-padding ux-ml-small"
						style="flex:auto;width:1rpx;">
						<text class="icon section-icon ux-color-blue">&#xe1b7;</text>
						<br>
						<text class="ux-text">雷达</text>
						<br>
						<text class="ux-text-small ux-opacity-8">实时预测附近经过的列车。</text>
						<br>
						<br>
						<view class="ux-text-right ux-mr-small">
							<!-- <text class="icon">&#xe5c8;</text>-->
						</view>
						<text class="ux-text-small">施工中 请静候佳音</text>
					</view>
				</view>
			</view>
			<br>
			<image class="ux-border-radius-large" src="/static/overlay/index_banner_1.png" style="width:100%;"
				mode="widthFix"></image>
		</view>
	</view>
</template>

<script>
	import uniGet from "@/scripts/req.js";
	import {
		loadDB,
		doQuery,
	} from "@/scripts/sqlite.js";
	import {
		KEYS_STRUCT_STATIONS,
		KEYS_STRUCT_TRAINS
	} from "@/scripts/config.js";
	export default {
		// Railgo Code
		data() {
			return {
				title: '海内存知己，天涯若比邻。',
				visit: 0,
				query: 0,
				items: ['暂无公告']
			};
		},
		mounted() {
			this.fetchData();
		},
		onShow() {
			// #ifdef APP
			plus.navigator.setStatusBarBackground('#114598');
			plus.navigator.setFullscreen(false);
			plus.navigator.showSystemNavigation();
			// #endif
		},
		async onLoad() {
			if (uni.getStorageSync("mode") == "local") {
				try {
					//await loadDB();
					uni.showToast({
						title: '加载数据库完成',
						position: 'bottom',
					})
					loadDB();
				} catch (error) {
					uni.setStorage({
						key: 'DBerror',
						data: error
					});
				}

			}
		},
		methods: {
			async fetchData() {
				try {
					const titleResponse = await uniGet('https://api.state.railgo.zenglingkun.cn/yiyan');
					this.title = titleResponse.data;

					const statsResponse = await uniGet('https://api.state.railgo.zenglingkun.cn/state');
					this.visit = statsResponse.data.visits;
					this.query = statsResponse.data.queries;

					const noticeResponse = await uniGet("https://api.state.railgo.zenglingkun.cn/notice");
					this.items = noticeResponse.data;

					await uniGet("https://api.state.railgo.zenglingkun.cn/visit");
				} catch (error) {
					console.error('Error fetching data:', error);
					this.title = '海内存知己，天涯若比邻。';
					this.visit = 0;
					this.query = 0;
					this.items = ['暂无公告'];
				}
			}
		}
	};
</script>

<style lang="scss">
	.section-icon {
		font-size: 50rpx;
	}

	.main {
		.box {
			background: #ffffff;
			//width: 95%;
			//height: 100px;
			//margin-top: 10px;
			//margin-left: 2.5%;
			//margin-right: 2.5%;
			//border-radius: 10px;
			box-sizing: border-box;
			font-family: "钉钉进步体";
			overflow: hidden;

			.desc {
				margin-left: 23px;
				font-size: 14px;
				position: absolute;
				margin-top: -5px;
			}

			.data {
				margin-left: 23px;
				font-size: 13px;
				position: absolute;
				margin-top: 13px;
				color: gray
			}
		}
	}

	.notice {
		width: 100%;
		height: 80rpx;
		line-height: 80rpx;
		background: #f9f9f9;
		margin: 0 auto;
		margin-top: 10px;
		display: flex;

		.left {
			width: 140rpx;
			display: flex;
			align-items: center;
			justify-content: center;

			.text {
				color: #114598;
				font-weight: 600;
				font-size: 28rpx;
			}
		}

		.center {
			flex: 1;

			swiper {
				height: 100%;

				&-item {
					height: 100%;
					font-size: 30rpx;
					color: #666;
					overflow: hidden;
					white-space: nowrap;
					text-overflow: ellipsis;
				}
			}
		}

		.right {
			width: 70rpx;
			display: flex;
			align-items: center;
			justify-content: center;
		}
	}
</style>