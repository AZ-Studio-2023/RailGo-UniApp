<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		<view class="ux-bg-primary" style="height: height: var(--status-bar-height);">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">车次查询</text>
			<br><br>
			<text class="ux-text-small ux-opacity-5">共查询到 {{this.showData.length}} 个车次</text>
		</view>
		<view class="ux-pl ux-pr ux-pb">
			<view class="ux-padding-small ux-h6 ux-text-center"
				style="background-color:#e9eef5;border:1px solid #114598;border-radius:10rpx;color:#114598;">
				<text class="ux-bold">信息仅供参考 请以铁路运营企业实际运用为准</text>
			</view>
			<br>
			<view class="ux-flex">
				<button class="ux-flex1 ux-mr-small ux-bg-primary ux-color-white" size="mini" @click="openSortMenu()">
					<view class="ux-flex ux-align-items-center ux-justify-content-center">
						<text class="icon">&#xe164;</text>&nbsp;排序
					</view>
				</button>
				<button class="ux-flex1 ux-ml-small ux-bg-primary ux-color-white" size="mini" @click="openFilterMenu()">
					<view class="ux-flex ux-align-items-center ux-justify-content-center">
						<text class="icon">&#xe06d;</text>&nbsp;筛选
					</view>
				</button>
			</view>
			<br>
			<navigator v-for="(item, index) in this.showData" :key="index"
				:url="'/pages/train/trainResult?keyword='+item.number+'&date='+this.date">
				<view class="ux-bg-white ux-border-radius ux-mb-small ux-flex">
					<view style="border-bottom-left-radius: 10rpx; border-top-left-radius:10rpx;"
						:style="'background-color:'+this.colorMap[item.number[0]]">
						&nbsp;&nbsp;
					</view>
					<view class="ux-flex ux-align-items-center ux-space-between ux-pr ux-pt ux-pb ux-pl-small"
						style="width:100%;">
						<view>
							<text class="consolas" style="font-size:45rpx;">
								{{item.timetable[item.fromPos].depart}}
							</text>
							<br>
							<text class="ux-text-small">
								{{item.timetable[item.fromPos].station}}
							</text>
						</view>
						<view class="ux-text-center">
							<text class="consolas" style="font-size:40rpx;">
								{{item.numberKind + item.numberFull.join("/").replace(item.numberKind, "").replace(item.numberKind, "")}}
							</text>
							<br>
							<view style="border-top: 0.1rpx solid #757575;width:30vw;margin: 5rpx 0;"></view>
							<text class="ux-text-small ux-opacity-5">
								{{item.passTime}}
							</text>
						</view>
						<view>
							<view class="ux-flex ux-align-items-start">
								<text class="consolas" style="font-size:45rpx;">
									{{item.timetable[item.toPos].arrive}}
								</text>
								<text v-if="item.timetable[item.toPos].day - item.timetable[item.fromPos].day!=0"
									style="font-size:20rpx">
									+{{item.timetable[item.toPos].day - item.timetable[item.fromPos].day}}
								</text>
							</view>
							<text class="ux-text-small">
								{{item.timetable[item.toPos].station}}
							</text>
						</view>
					</view>
				</view>
			</navigator>
			<br>
			<view class="ux-flex ux-row ux-justify-content-center">
				<text class="ux-text-small ux-opacity-4">—— 数据来源: RailGo.Parser ——</text>
			</view>
		</view>
	</view>
	<uni-popup ref="error_noky" border-radius="10rpx 10rpx 0 0">
		<uni-popup-dialog title="未查询到符合条件的车次" content="发站或到站中有车站不办理客运业务或无列车停靠。" @close="back"
			@confirm="back"></uni-popup-dialog>
	</uni-popup>
	<uni-popup ref="error_nosuch" border-radius="10rpx 10rpx 0 0">
		<uni-popup-dialog title="未查询到符合条件的车次" content="两站间在选定日期不存在直达列车。" @close="back"
			@confirm="back"></uni-popup-dialog>
	</uni-popup>
	<uni-popup ref="menu_sort" border-radius="10rpx 10rpx 0 0">
		<view class="popup-content">
			<view class="ux-bg-white ux-padding ux-border-radius" style="width:70vw;">
				<text class="ux-h4">车次排序</text>
				<br>
				<radio-group class="ux-mt-small" @change="radioSortChange">
					<radio color="#114598" value="departure" class="ux-mr ux-mt-small"
						:checked="this.sortState=='departure'">
						<text class="ux-text-small">按发车时间</text>
					</radio>
					<br>
					<radio color="#114598" value="arrival" class="ux-mr ux-mt-small"
						:checked="this.sortState=='arrival'">
						<text class="ux-text-small">按到达时间</text>
					</radio>
					<br>
					<radio color="#114598" value="speed" class="ux-mr ux-mt-small" :checked="this.sortState=='speed'">
						<text class="ux-text-small">按速度</text>
					</radio>
				</radio-group>
			</view>
		</view>
	</uni-popup>
	<uni-popup ref="menu_filter" border-radius="10rpx 10rpx 0 0">
		<view class="popup-content">
			<view class="ux-bg-white ux-padding ux-border-radius" style="width:70vw;">
				<text class="ux-h4">车次筛选</text>
				<br>
				<checkbox-group class="ux-mt-small" @change="radioFilterChange">
					<view class="ux-flex ux-space-between">
						<checkbox color="#114598" value="G" class="ux-mr ux-mt-small"
							:checked="this.filterTypeState.includes('G')">
							<text class="ux-text-small">高速</text>
						</checkbox>
						<checkbox color="#114598" value="D" class="ux-mr ux-mt-small"
							:checked="this.filterTypeState.includes('D')">
							<text class="ux-text-small">动车</text>
						</checkbox>
						<checkbox color="#114598" value="C" class="ux-mr ux-mt-small"
							:checked="this.filterTypeState.includes('C')">
							<text class="ux-text-small">城际</text>
						</checkbox>
					</view>
					<view class="ux-flex ux-space-between">
						<checkbox color="#114598" value="Z" class="ux-mr ux-mt-small"
							:checked="this.filterTypeState.includes('Z')">
							<text class="ux-text-small">直达</text>
						</checkbox>
						<checkbox color="#114598" value="T" class="ux-mr ux-mt-small"
							:checked="this.filterTypeState.includes('T')">
							<text class="ux-text-small">特快</text>
						</checkbox>
						<checkbox color="#114598" value="K" class="ux-mr ux-mt-small"
							:checked="this.filterTypeState.includes('K')">
							<text class="ux-text-small">快速</text>
						</checkbox>
					</view>
					<view class="ux-flex ux-space-between">
						<checkbox color="#114598" value="12345678" class="ux-mr ux-mt-small"
							:checked="this.filterTypeState.includes('1')">
							<text class="ux-text-small">普客</text>
						</checkbox>
						<checkbox color="#114598" value="S" class="ux-mr ux-mt-small"
							:checked="this.filterTypeState.includes('S')">
							<text class="ux-text-small">市域</text>
						</checkbox>
						<checkbox color="#114598" value="LY" class="ux-mr ux-mt-small"
							:checked="this.filterTypeState.includes('L')">
							<text class="ux-text-small">其他</text>
						</checkbox>
					</view>
				</checkbox-group>
			</view>
		</view>
	</uni-popup>
</template>

<script>
	import {
		queryMainKey
	} from "@/scripts/jsonDB.js";
	import {
		doQuery,
	} from "@/scripts/sqlite.js";
	import {
		toRaw
	} from "@vue/reactivity";
	import {
		KEYS_STRUCT_STATIONS,
		KEYS_STRUCT_TRAINS,
		TRAIN_KIND_COLOR_MAP
	} from "@/scripts/config.js";
import uniGet from "@/scripts/req";

	export default {
		data() {
			return {
				"keyword": "",
				"date": "",
				"from": "",
				"to": "",
				"data": [],
				"showData": [],
				"colorMap": TRAIN_KIND_COLOR_MAP,
				"sortState": "departure",
				"filterTypeState": "GDCZTK12345678SLY"
			}
		},
		onLoad(options) {
			this.from = options.from;
			this.to = options.to;
			this.error = "";
			this.date = options.date;
			const c = uni.getStorageSync("search");
			uni.setStorage({
				key: 'search',
				data: c+1
			});
			try{
				this.fillInData();
			} catch(error){
				uni.setStorage({
				"key": "DBerror",
				"data": error.message
				})
			}
			
		},
		onShow() {
			// #ifdef APP
			plus.navigator.setStatusBarBackground('#114598');
			// #endif
		},
		methods: {
			back: function() {
				uni.navigateBack()
			},
			fillInData: async function() {
				const mode = uni.getStorageSync("mode"); 
				if (mode == "network") {
					// --- 网络模式逻辑 ---
					try {
						uni.showLoading({
							title: "加载中"
						});
						const resp = await uniGet(`https://data.railgo.zenglingkun.cn/api/train/sts_query?from=${this.from}&to=${this.to}&date=${this.date}`);
						const result = resp.data;
			
						if (result.error) {
							uni.hideLoading();
							const c = uni.getStorageSync("search");
							uni.setStorage({
								key: 'search',
								data: c - 1
							});
							this.$refs.error_noky.open();
							return;
						}
			
						this.data = result;
						this.showData = this.data;
						this.radioSortChange({
							detail: {
								value: "departure"
							}
						});
			
						if (toRaw(this.data).length == 0) {
							uni.hideLoading();
							const c = uni.getStorageSync("search");
							uni.setStorage({
								key: 'search',
								data: c - 1
							});
							this.$refs.error_nosuch.open();
							return;
						}
						uni.hideLoading();
			
					} catch (error) {
						uni.hideLoading();
						console.error("数据加载失败", error);
						const c = uni.getStorageSync("search");
						uni.setStorage({
							key: 'search',
							data: c - 1
						});
						uni.showToast({
							title: "加载失败",
							duration: 1000
						});
					}
				} else {
					// --- 本地模式逻辑 ---
					try {
						uni.showLoading({
							title: "加载中"
						});
						let fromStn = toRaw(await doQuery("SELECT trainList FROM stations WHERE telecode='" + this.from +
							"'", ["trainList"]))[0];
						let toStn = toRaw(await doQuery("SELECT trainList FROM stations WHERE telecode='" + this.to + "'",
							["trainList"]))[0];
			
						if (fromStn.trainList.length == 0 || toStn.trainList.length == 0) {
							uni.hideLoading();
							this.$refs.error_noky.open();
							return;
						}
			
						let d = [];
						let all = toRaw(await doQuery(
							"SELECT code, number, numberFull, numberKind, timetable, rundays FROM trains WHERE number IN ('" +
							fromStn.trainList
							.filter((i) => {
								return toStn.trainList.includes(i)
							}).join("','") + "')", ["code", "number", "numberFull", "numberKind", "timetable",
								"rundays"
							]));
			
						all.forEach((k) => {
							let fromPos = -1;
							let toPos = -1;
							if (!k.rundays.includes(this.date)) {
								return;
							}
							for (var i = 0; i < k.timetable.length; i++) {
								if (k.timetable[i].stationTelecode == this.from) {
									fromPos = i;
								}
								if (k.timetable[i].stationTelecode == this.to) {
									toPos = i;
									break;
								}
							}
							if (fromPos != -1) {
								k.fromPos = fromPos;
								k.toPos = toPos;
								k.showFlag = true;
								k.passTime = this.calculateTimeDifference(k.timetable[k.fromPos].depart, k
									.timetable[k.toPos].arrive, k.timetable[k.toPos].day - k.timetable[k
										.fromPos].day);
								this.data.push(k);
							}
						}, this);
			
						this.showData = this.data;
						this.radioSortChange({
							detail: {
								value: "departure"
							}
						});
			
						if (toRaw(this.data).length == 0) {
							uni.hideLoading();
							this.$refs.error_nosuch.open();
							return;
						}
						uni.hideLoading();
			
					} catch (error) {
						uni.hideLoading();
						// 注意：plus.nativeUI.alert 仅在 H5+ 或 App 平台可用
						if (typeof plus !== 'undefined' && plus.nativeUI) {
							plus.nativeUI.alert(error);
						}
						console.error("数据加载失败", error);
						uni.showToast({
							title: "加载失败",
							duration: 1000
						});
					}
				}
			},

			calculateTimeDifference: function(startTime, endTime, daysLater) {
				const parseTime = (timeStr) => {
					const [hours, minutes] = timeStr.split(':').map(Number);
					return {
						hours,
						minutes
					};
				};
				const start = parseTime(startTime);
				const end = parseTime(endTime);
				const startTotalMinutes = start.hours * 60 + start.minutes;
				const endTotalMinutes = end.hours * 60 + end.minutes + (daysLater * 24 * 60);
				let differenceMinutes = endTotalMinutes - startTotalMinutes;
				if (differenceMinutes < 0) differenceMinutes += 24 * 60;
				const hours = Math.floor(differenceMinutes / 60);
				const minutes = differenceMinutes % 60;
				return `${hours}h${minutes.toString().padStart(2, '0')}m`;
			},
			openSortMenu: function() {
				this.$refs.menu_sort.open();
			},
			openFilterMenu: function() {
				this.$refs.menu_filter.open();
			},
			radioSortChange: function(e) {
				this.sortState = e.detail.value;
				switch (e.detail.value) {
					case "speed":
						this.showData = this.data.sort((a, b) => {
							if (a.passTime > b.passTime) {
								return 1;
							}
							if (a.passTime < b.passTime) {
								return -1;
							}
							return 0;
						});
						break;

					case "departure":
						this.showData = this.data.sort((a, b) => {
							if (a.timetable[a.fromPos].depart > b.timetable[b.fromPos].depart) {
								return 1;
							}
							if (a.timetable[a.fromPos].depart < b.timetable[b.fromPos].depart) {
								return -1;
							}
							return 0;
						});
						break;

					case "arrival":
						this.showData = this.data.sort((a, b) => {
							if (a.timetable[a.toPos].arrive > b.timetable[b.toPos].arrive) {
								return 1;
							}
							if (a.timetable[a.toPos].arrive < b.timetable[b.toPos].arrive) {
								return -1;
							}
							return 0;
						});
						break;

					default:
						console.log("WHAT THE FUCK R U DOING?");
				}
				this.$refs.menu_sort.close();
			},
			radioFilterChange: function(e) {
				this.filterTypeState = e.detail.value.join("");
				this.showData = this.data.filter((i) => {
					return e.detail.value.join("").includes(i.number.charAt(0));
				});
			}
		}
	}
</script>