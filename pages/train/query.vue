<template>
	<view class="ux-bg-grey5 page">
		<!-- headers begin -->
		<view class="ux-bg-primary status-bar"></view>
		<view class="ux-pl ux-pr ux-pt">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">车次查询</text>
		</view>
		<!-- headers end -->
		<view class="ux-pl ux-pr ux-pb">
			<view class="ux-flex ux-justify-content-center">
				<uv-tabs :list="topTabList" lineWidth="60" lineColor="#114598" :activeStyle="{
						color: '#303133',
						fontWeight: 'bold',
						transform: 'scale(1.05)'
			    	}" :inactiveStyle="{
						color: '#606266',
						transform: 'scale(1)'
					}" itemStyle="height: 34px;padding-left:30px;padding-right:30px;" class="ux-mt-small"
					@click="tabChange"></uv-tabs>
			</view>
			<view v-if="selectIndex==0">
				<view class="ux-pt-small ux-bg-grey5">
					<text class="ux-h6">车次</text>
					<view style="position:relative;">
						<view
							class="ux-bg-white ux-border-radius-small ux-padding-small ux-flex ux-align-items-center ux-mt-small ux-mb-small">
							<text class="icon">&#xe8b6;</text>
							&nbsp;
							<input type="text" class="ux-form-input" placeholder="车次号码" style="width:100%;margin:0 0;"
								@input="inputData" v-model="keyword" />
						</view>
						<view v-if="shouldShowPlacehold()"
							style="position:absolute;z-index:114514;margin-top:-0.78125rem;width:100%;"
							class="ux-bg-white ux-box-shadow ux-text">
							<scroll-view scroll-y="true" style="max-height:7rem;">
								<view v-for="(item,index) in placeholderData" :key="index"
									class="ux-flex ux-space-between ux-align-items-center"
									style="padding:0.4rem 0.5rem;" hover-class="ux-bg-grey5"
									@click='inputPlacehold(item)'>
									<view class="ux-flex ux-space-around">
										<view style="font-size:0.96rem;">
											{{item.numberFull.join("/")}}
										</view>
									</view>
									<view class="ux-opacity-5 ux-text-small">
										{{item.timetable[0].station}} ⋙
										{{item.timetable[item.timetable.length-1].station}}
									</view>
								</view>
							</scroll-view>
						</view>
					</view>

				</view>
			</view>
			<view v-if="selectIndex==1">
				<view class="ux-pt-small ux-bg-grey5">
					<view class="ux-flex ux-space-around ux-align-items-center">
						<view class="ux-mr-small" style="width:100%">
							<text class="ux-h6">发站</text>
							<navigator url="/pages/station/commonSelect?resultPlace=train_sts_fieldA">
								<view
									class="ux-bg-white ux-border-radius-small ux-padding-small ux-flex ux-align-items-center ux-mt-small ux-mb-small">
									{{this.stsSelectionA}}
								</view>
							</navigator>
						</view>
						<view>
							<text class="ux-h6"></text>
							<br>
							<view class="ux-flex ux-align-items-center ux-justify-content-center"
								style="border:5rpx solid #114598;border-radius:50%;width:50rpx;height:50rpx;"
								hover-class="ux-bg-grey4" @click="swapSts">
								<text class="icon ux-color-primary ux-text-bold"
									style="font-size:40rpx;">&#xe8be;</text>
							</view>
						</view>
						<view class="ux-pl-small" style="width:100%">
							<text class="ux-h6">到站</text>
							<navigator url="/pages/station/commonSelect?resultPlace=train_sts_fieldB">
								<view
									class="ux-bg-white ux-border-radius-small ux-padding-small ux-flex ux-align-items-center ux-mt-small ux-mb-small">
									{{this.stsSelectionB}}
								</view>
							</navigator>
						</view>
					</view>
				</view>
			</view>
			<text class="ux-h6">日期</text>
			<view class="ux-bg-white ux-border-radius-small ux-mb-small ux-mt-small">
				<uni-datetime-picker type="date" v-model="today" start="2007-04-18" end="2099-01-01"
					@change="inputDate" />
			</view>
			<br>
			<button type="primary" style="background-color:#114598;color:#ffffff;" hover-class="ux-tap"
				@click="jumpToResult()">查询</button>
			<br>
			<view class="ux-text-center ux-padding-small ux-mb ux-h6"
				style="background-color:#e9eef5;border:1px solid #114598;border-radius:10rpx;color:#114598;">
				<text class="ux-bold">信息仅供参考 请以铁路运营企业实际运用为准</text>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		query
	} from "@/scripts/jsonDB.js";
	import {
		doQuery,
	} from "@/scripts/sqlite.js";
	import {
		KEYS_STRUCT_STATIONS,
		KEYS_STRUCT_TRAINS
	} from "@/scripts/config.js";
	import {
		toRaw
	} from "@vue/reactivity";
	import {
		TRAIN_KIND_COLOR_MAP
	} from "@/scripts/config.js";
	export default {
		data() {
			return {
				"keyword": "",
				"target": "",
				"colorMap": TRAIN_KIND_COLOR_MAP,
				"date": this.getToday(),
				"today": this.getTodayRaw(),
				"placeholderData": [],
				"placeholderCollapsed": false,
				"topTabList": [{
					name: '车次',
				}, {
					name: '站站',
				}],
				"selectIndex": 0,
				"stsSelectionA": "",
				"stsSelectionB": ""
			}
		},
		onShow() {
			let selectionA = uni.getStorageSync("train_sts_fieldA");
			let selectionB = uni.getStorageSync("train_sts_fieldB");
			if (!selectionA || !selectionB) {
				uni.setStorageSync("train_sts_fieldA", {
					name: "北京",
					telecode: "BJP"
				});
				uni.setStorageSync("train_sts_fieldB", {
					name: "上海",
					telecode: "SHH"
				});
				this.stsSelectionA = "北京";
				this.stsSelectionA = "上海";
			}
			this.stsSelectionA = selectionA.name;
			this.stsSelectionB = selectionB.name;
		},
		methods: {
			back: function() {
				uni.navigateBack()
			},
			jumpToResult: function() {
				if (this.selectIndex == 0) {
					if (this.keyword == "" || this.date == "") {
						uni.showToast({
							icon: "none",
							title: "不允许输入空值"
						});
						return;
					}
					uni.navigateTo({
						url: "/pages/train/trainResult?keyword=" + this.target + "&date=" + this.date
					});
				} else {
					uni.navigateTo({
						url: "/pages/train/stsResult?from=" + uni.getStorageSync("train_sts_fieldA").telecode +
							"&to=" + uni.getStorageSync("train_sts_fieldB").telecode + "&date=" + this.date
					});
				}
			},
			inputData: async function(e) {
				this.keyword = e.detail.value;
				if (this.keyword.length >= 2) {
					if (this.keyword[0] in ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]) {
						this.placeholderData = toRaw(await doQuery(
							"SELECT code, numberFull, timetable FROM trains WHERE numberFull LIKE '%\"_" +
							this.keyword + "%\"%' OR numberFull LIKE '%\"" + this.keyword + "%\"'",
							["code", "numberFull", "timetable"]));
					} else {
						this.placeholderData = toRaw(await doQuery(
							"SELECT code, numberFull, timetable FROM trains WHERE numberFull LIKE '%" +
							this.keyword + "%'", ["code", "numberFull", "timetable"]));
					}
					this.placeholderData = this.placeholderData.sort((a, b) => parseInt(a.numberFull.join("/")
						.match(
							/\d+/)[0]) - parseInt(b.numberFull
						.join("/").match(/\d+/)[0]));
					this.placeholderCollapsed = false;
				}
			},
			inputDate: function(e) {
				// replace不用RegExp只会替换一次
				this.date = e.replace("-", "").replace("-", "");
			},
			shouldShowPlacehold: function() {
				return this.keyword.length >= 2 && !this.placeholderCollapsed && this.placeholderData.length != 0;
			},
			inputPlacehold: function(ph) {
				this.keyword = ph.numberFull.join("/");
				this.target = ph.code;
				this.placeholderCollapsed = true;
			},
			swapSts: function(e) {
				let selectionA = uni.getStorageSync("train_sts_fieldA");
				let selectionB = uni.getStorageSync("train_sts_fieldB");
				uni.setStorageSync("train_sts_fieldA", selectionB);
				uni.setStorageSync("train_sts_fieldB", selectionA);
				this.stsSelectionA = selectionB.name;
				this.stsSelectionB = selectionA.name;
			},
			tabChange: function(e) {
				this.selectIndex = e.index;
			},
			getToday: function() {
				const date = new Date();
				return `${date.getFullYear()}${('0' + (date.getMonth() + 1)).slice(-2)}${('0' + date.getDate()).slice(-2)}`;
			},
			getTodayRaw: function() {
				const date = new Date();
				return `${date.getFullYear()}-${('0' + (date.getMonth() + 1)).slice(-2)}-${('0' + date.getDate()).slice(-2)}`;
			}
		}
	}
</script>

<style>

</style>