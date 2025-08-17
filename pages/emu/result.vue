<template>
	<view class="ux-bg-grey5 page">
		<!-- headers begin -->
		<view class="ux-bg-primary status-bar"></view>
		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">{{keyword}}</text>
		</view>
		<!-- headers end -->
		<view class="ux-pl ux-pr ux-pb">
			<view class="ux-text-center ux-padding-small ux-mb ux-h6 ux-text-center"
				style="background-color:#e9eef5;border:1px solid #114598;border-radius:10rpx;color:#114598;">
				<text class="ux-bold">信息仅供参考 请以铁路运营企业实际运用为准</text>
			</view>
			<uni-section style="background-color:#eeeeee;margin-left:-1.2vh;" type="line"
				title="点击车次或车组可查询对应车辆和车次的担当"></uni-section>
			<uni-table border stripe>
				<uni-tr>
					<uni-th align="center">担当日</uni-th>
					<uni-th align="center">车次</uni-th>
					<uni-th align="left">车组</uni-th>
				</uni-tr>
				<uni-tr v-for="(item,index) in data" :key="index">
					<uni-td>{{item.runDate}}</uni-td>
					<uni-td>
						<navigator :url="getTrainNavigatorURL(item)">
							{{item.trainNum.join("/")}}
						</navigator>
					</uni-td>
					<uni-td>
						<navigator :url="getCarNavigatorURL(item, 0)">
							{{item.trainCode[0]}}
						</navigator>
						<view v-if="item.trainCode[1]!=undefined">
							<navigator :url="getCarNavigatorURL(item, 1)">
								{{item.trainCode[1]}}
							</navigator>
						</view>
					</uni-td>
				</uni-tr>
			</uni-table>
			<br>
			<view class="ux-flex ux-row ux-justify-content-center">
				<text class="ux-text-small ux-opacity-4">—— 数据来源: CR-EMU-Tracker ——</text>
			</view>
		</view>
	</view>
</template>

<script>
	import uniGet from "@/scripts/req.js";
	export default {
		data() {
			return {
				"keyword": "",
				"future": true,
				"data": []
			}
		},
		onLoad(options) {
			this.keyword = options.keyword;
			this.future = options.future;
			this.fillInData();
			uni.showLoading({
				title: "加载中"
			});
		},
		onShow() {
			plus.navigator.setStatusBarBackground('#eeeeee');
		},
		methods: {
			back: function() {
				uni.navigateBack()
			},
			fillInData: async function() {
				try {
					const response = await uniGet("https://crtracker.azteam.cn/api/query?keyword=" + this.keyword +
						"&future=" + this.future);
					this.data = response.data.data;
					uni.hideLoading();
				} catch (error) {
					console.error("数据加载失败", error);
					uni.showToast({
						title: "加载失败",
						duration: 1000
					});
				}
			},
			getTrainNavigatorURL: function(item) {
				return '/pages/emu/result?keyword=' + item.trainNum[0] + '&future=' + this.future;
			},
			getCarNavigatorURL: function(item, index) {
				return '/pages/emu/result?keyword=' + item.trainCode[index] + '&future=' + this.future;
			}
		}
	}
</script>