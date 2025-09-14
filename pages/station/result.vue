<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		<view class="ux-bg-primary" style="height:  var(--status-bar-height);">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
		</view>
		<view class="ux-padding">
			<view class="ux-bg-white ux-border-radius">
				<view class="ux-pl ux-pr ux-pb-small ux-pt ux-flex ux-align-items-center ux-justify-content-center">
					<view class="ux-flex ux-align-items-center">
						<view v-for="(tag,i) in data.type" :key="i">
							<text class="ux-badge ux-color-white ux-mr-small"
								:style="'background-color:'+badgeFlag[tag]">{{tag}}</text>
						</view>
					</view>
					<view>
						<text class="ux-h3">{{data.name}}站</text><br>
						<text class="ux-text-small">{{data.pinyin}} Station</text>
					</view>
				</view>
				<view class="ux-flex ux-space-between ux-color-white ux-pt-small ux-pb-small ux-pl ux-pr ux-text-small"
					style="border-bottom-left-radius:10rpx;border-bottom-right-radius:10rpx;"
					:style="'background-color:'+(Array.isArray(data.type) && data.type.includes('客') ? '#114598' : '#eeba67')">
					<text>{{data.pinyinTriple}}/-{{data.telecode}}</text>
					<text>{{data.bureau}} {{data.belong}}辖</text>
				</view>
			</view>

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

			<view class="ux-pt" v-if="selectIndex==0">
				<view v-if="trains.length!=0">
					<view class="ux-flex">
						<button class="ux-flex1 ux-mr-small ux-bg-primary ux-color-white" size="mini"
							@click="openSortMenu()">
							<view class="ux-flex ux-align-items-center ux-justify-content-center">
								<text class="icon">&#xe164;</text>&nbsp;排序
							</view>
						</button>
						<button class="ux-flex1 ux-ml-small ux-bg-primary ux-color-white" size="mini"
							@click="openFilterMenu()">
							<view class="ux-flex ux-align-items-center ux-justify-content-center">
								<text class="icon">&#xe06d;</text>&nbsp;筛选
							</view>
						</button>
					</view>
					<view class="ux-mt-small">
						<text class="ux-text-small ux-opacity-5">共查询到 {{showTrains.length}} 个车次</text>
						<br>
					</view>
					<navigator v-for="(item,index) in showTrains" :key="index"
											:url="'/pages/train/trainResult?keyword='+(item.number || item.code)+'&date='+getToday()">
						<view class="ux-bg-white ux-border-radius ux-mt-small ux-flex">
							<view style="border-bottom-left-radius: 10rpx; border-top-left-radius:10rpx;"
								:style="'background-color:'+colorMap[item.number.charAt(0)]">
								&nbsp;&nbsp;
							</view>
							<view class="ux-flex ux-align-items-center ux-space-between ux-pr ux-pt ux-pb ux-pl-small"
								style="width:100%;">
								<view style="width:calc(100% - 60rpx);">
									<view class="ux-flex ux-space-between">
										<view>
											<text class="consolas" style="font-size:40rpx;">
												{{item.numberKind}}{{item.numberFull.join("/").replace(new RegExp(item.numberKind, 'g'), "")}}
											</text>
											<br>
											<text class="ux-text-small">{{getStationName(item, "from")}} ⋙ {{getStationName(item, "to")}}</text>
										</view>
										<view class="ux-flex" style="padding-top:8rpx;">
											<view class="ux-text-center ux-mr-small" style="min-width:80rpx">
												<text class="ux-text">{{getArriveTime(item) || '--:--'}}</text>
												<br>
												<text class="ux-text-small ux-opacity-5">到达</text>
											</view>
											<view class="ux-text-center ux-mr-small" style="min-width:80rpx">
												<text class="ux-text">{{getDepartTime(item) || '--:--'}}</text>
												<br>
												<text class="ux-text-small ux-opacity-5">出发</text>
											</view>
											<view class="ux-text-center">
												<text class="ux-text">{{getStopTime(item) || '-'}}</text>
												<br>
												<text class="ux-text-small ux-opacity-5">停车</text>
											</view>
										</view>

									</view>
								</view>
								<text class="ux-text"><text class="icon">&#xe5c8;</text></text>
							</view>
						</view>
					</navigator>
				</view>
				<view v-if="trains.length==0">
					<view class="ux-padding ux-text-center">本站不办理客运业务或无列车停靠</view>
				</view>
				<br>
				<view class="ux-flex ux-row ux-justify-content-center">
					<text class="ux-text-small ux-opacity-4">—— 数据来源: RailGo.Parser ——</text>
				</view>
			</view>

			<view class="ux-padding ux-text-center" v-if="selectIndex==2">
				<text>暂未开放，敬请期待</text>
			</view>
		</view>
	</view>
	<uni-popup ref="menu_sort" border-radius="10rpx 10rpx 0 0">
		<view class="popup-content">
			<view class="ux-bg-white ux-padding ux-border-radius" style="width:70vw;">
				<text class="ux-h4">车次排序</text>
				<br>
				<radio-group class="ux-mt-small" @change="radioSortChange">
					<radio color="#114598" value="departure" class="ux-mr ux-mt-small"
						:checked="sortState=='departure'">
						<text class="ux-text-small">按发车时间</text>
					</radio>
					<br>
					<radio color="#114598" value="arrival" class="ux-mr ux-mt-small"
						:checked="sortState=='arrival'">
						<text class="ux-text-small">按到达时间</text>
					</radio>
					<br>
					<radio color="#114598" value="stop" class="ux-mr ux-mt-small" :checked="sortState=='stop'">
						<text class="ux-text-small">按停车时长</text>
					</radio>
				</radio-group>
			</view>
		</view>
	</uni-popup>
	<uni-popup ref="menu_filter" border-radius="10rpx 10rpx 0 0">
		<view class="popup-content">
			<view class="ux-bg-white ux-padding ux-border-radius" style="width:70vw;">
				<text class="ux-h4">车种筛选</text>
				<br>
				<checkbox-group class="ux-mt-small" @change="radioFilterChange">
					<view class="ux-flex ux-space-between">
						<checkbox color="#114598" value="G" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('G')">
							<text class="ux-text-small">高速</text>
						</checkbox>
						<checkbox color="#114598" value="D" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('D')">
							<text class="ux-text-small">动车</text>
						</checkbox>
						<checkbox color="#114598" value="C" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('C')">
							<text class="ux-text-small">城际</text>
						</checkbox>
					</view>
					<view class="ux-flex ux-space-between">
						<checkbox color="#114598" value="Z" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('Z')">
							<text class="ux-text-small">直达</text>
						</checkbox>
						<checkbox color="#114598" value="T" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('T')">
							<text class="ux-text-small">特快</text>
						</checkbox>
						<checkbox color="#114598" value="K" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('K')">
							<text class="ux-text-small">快速</text>
						</checkbox>
					</view>
					<view class="ux-flex ux-space-between">
						<checkbox color="#114598" value="12345678" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('1')">
							<text class="ux-text-small">普客</text>
						</checkbox>
						<checkbox color="#114598" value="S" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('S')">
							<text class="ux-text-small">市域</text>
						</checkbox>
						<checkbox color="#114598" value="LY" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('L')">
							<text class="ux-text-small">其他</text>
						</checkbox>
					</view>
				</checkbox-group>
				<uv-divider></uv-divider>
				<text class="ux-h4">到发筛选</text>
				<br>
				<checkbox-group class="ux-mt-small" @change="radioSourceChange">
					<view class="ux-flex ux-space-between">
						<checkbox color="#114598" value="P" class="ux-mr ux-mt-small"
							:checked="filterSourceState.includes('P')">
							<text class="ux-text-small">过路</text>
						</checkbox>
						<checkbox color="#114598" value="D" class="ux-mr ux-mt-small"
							:checked="filterSourceState.includes('D')">
							<text class="ux-text-small">本站始发</text>
						</checkbox>
						<checkbox color="#114598" value="A" class="ux-mr ux-mt-small"
							:checked="filterSourceState.includes('A')">
							<text class="ux-text-small">本站终到</text>
						</checkbox>
					</view>
				</checkbox-group>
			</view>
		</view>
	</uni-popup>
</template>

<script>
	import {
		doQuery
	} from "@/scripts/sqlite.js";
	import {
		toRaw
	} from "@vue/reactivity";
	import {
		KEYS_STRUCT_STATIONS,
		KEYS_STRUCT_TRAINS,
		TRAIN_KIND_COLOR_MAP
	} from '@/scripts/config.js';
	import uniGet from "../../scripts/req";

	export default {
		data() {
			return {
				keyword: "",
				data: {},
				badgeFlag: {
					"客": "#114598",
					"货": "#eeba67",
					"高": "#c0392b",
					"行": "#459811",
					"运": "#5499c7"
				},
				topTabList: [{
					name: '车次',
				}, {
					name: '大屏',
				}, {
					name: '路线'
				}],
				selectIndex: 0,
				trains: [],
				showTrains: [],
				colorMap: TRAIN_KIND_COLOR_MAP,
				filterTypeState: "GDCZTK12345678SLY",
				filterSourceState: "PDA",
				sortState: "departure"
			}
		},
		onLoad(options) {
			this.keyword = options.keyword;
			const mode = uni.getStorageSync("mode");
			const c = uni.getStorageSync("search");
			uni.setStorage({
				key: 'search',
				data: c + 1
			});
			this.fillInData(mode);
		},
		methods: {
			back: function() {
				uni.navigateBack();
			},
			fillInData: async function(mode) {
				uni.showLoading({
					title: "加载中"
				});

				if (mode == "network") {
					try {
						const resp = await uniGet(`https://data.railgo.zenglingkun.cn/api/station/query?telecode=${this.keyword}`);
						const result = resp.data;
						if (result.error) {
							uni.showToast({
								title: '车站不存在',
								icon: 'error'
							});
							const c = uni.getStorageSync("search");
							uni.setStorage({
								key: 'search',
								data: c - 1
							});
							uni.hideLoading();
							uni.navigateBack();
							return;
						}
						this.data = result.data || {};
						this.trains = result.trains || [];
					} catch (error) {
						uni.showToast({
							title: '加载失败',
							icon: 'error'
						});
						console.error("车站数据加载失败", error);
						// #ifdef APP-PLUS
						plus.nativeUI.alert("调试错误：" + error);
						// #endif
					}
				} else { // 本地模式
					try {
						this.data = toRaw((await doQuery("SELECT * FROM stations WHERE telecode='" + this.keyword + "'", KEYS_STRUCT_STATIONS))[0]);
						if (this.data.trainList && this.data.trainList.length > 0) {
							this.trains = toRaw(await doQuery(
								"SELECT code, number, numberFull, numberKind, timetable FROM trains WHERE number IN ('" +
								this.data.trainList.join("','") + "')", ["code", "number", "numberFull", "numberKind",
									"timetable"
								]));
							this.trains.forEach((item, index) => {
								item.indexStopThere = item.timetable.findIndex((tt) => {
									return tt.stationTelecode == this.keyword;
								});
								this.trains[index] = item;
							});
						} else {
							this.trains = [];
						}
					} catch (error) {
						console.error("本地数据加载失败", error);
					}
				}
				this.radioSortChange({
					detail: {
						value: "departure"
					}
				});
				uni.hideLoading();
			},
			tabChange: function(e) {
				this.selectIndex = e.index;
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
			        case "stop":
			            this.showTrains = this.trains.sort((a, b) => {
			                const stopTimeA = (a.timetable && a.indexStopThere !== -1) ? (a.timetable[a.indexStopThere].stopTime || 0) : 0;
			                const stopTimeB = (b.timetable && b.indexStopThere !== -1) ? (b.timetable[b.indexStopThere].stopTime || 0) : 0;
			                return parseInt(stopTimeA) - parseInt(stopTimeB);
			            });
			            break;
			        case "departure":
			            this.showTrains = this.trains.sort((a, b) => {
			                const departA = (a.timetable && a.indexStopThere !== -1) ? (a.timetable[a.indexStopThere].depart || '') : '';
			                const departB = (b.timetable && b.indexStopThere !== -1) ? (b.timetable[b.indexStopThere].depart || '') : '';
			                return departA.localeCompare(departB);
			            });
			            break;
			        case "arrival":
			            this.showTrains = this.trains.sort((a, b) => {
			                const arriveA = (a.timetable && a.indexStopThere !== -1) ? (a.timetable[a.indexStopThere].arrive || '') : '';
			                const arriveB = (b.timetable && b.indexStopThere !== -1) ? (b.timetable[b.indexStopThere].arrive || '') : '';
			                return arriveA.localeCompare(arriveB);
			            });
			            break;
			        default:
			            console.log("排序值无效");
			    }
			    this.$refs.menu_sort.close();
			},
			radioFilterChange: function(e) {
				this.filterTypeState = e.detail.value.join("");
				this.showTrains = this.trains.filter((i) => {
					return this.filterTypeState.includes(i.number.charAt(0));
				});
			},
			radioSourceChange: function(e) {
				this.filterSourceState = e.detail.value.join("");
				this.showTrains = this.trains.filter((i) => {
					const fromCode = (i.fromStation && i.fromStation.stationTelecode) || (i.timetable && i.timetable[0] && i.timetable[0].stationTelecode) || '';
					const toCode = (i.toStation && i.toStation.stationTelecode) || (i.timetable && i.timetable[i.timetable.length - 1] && i.timetable[i.timetable.length - 1].stationTelecode) || '';

					return (
						(fromCode !== this.keyword && toCode !== this.keyword && this.filterSourceState.includes("P")) ||
						(fromCode === this.keyword && this.filterSourceState.includes("D")) ||
						(toCode === this.keyword && this.filterSourceState.includes("A"))
					);
				});
			},
			getToday: function() {
				const date = new Date();
				return `${date.getFullYear()}${('0' + (date.getMonth() + 1)).slice(-2)}${('0' + date.getDate()).slice(-2)}`;
			},
			getStationName(item, type) {
				if (type === "from") {
					return item.fromStation?.station || item.timetable?.[0]?.station || '';
				} else {
					return item.toStation?.station || item.timetable?.[item.timetable.length - 1]?.station || '';
				}
			},
			getArriveTime(item) {
			    if (item.arrive) {
			        return item.arrive;
			    }
			    // Add a check to ensure item.timetable is not undefined
			    if (item.timetable) {
			        const stop = item.timetable.find(tt => tt.stationTelecode === this.keyword);
			        return stop ? stop.arrive : '--:--';
			    }
			    return '--:--'; // Return a default value if timetable is missing
			},
			
			getDepartTime(item) {
			    if (item.depart) {
			        return item.depart;
			    }
			    // Add a check to ensure item.timetable is not undefined
			    if (item.timetable) {
			        const stop = item.timetable.find(tt => tt.stationTelecode === this.keyword);
			        return stop ? stop.depart : '--:--';
			    }
			    return '--:--'; // Return a default value
			},
			
			getStopTime(item) {
			    if (item.stopTime) {
			        return item.stopTime;
			    }
			    // Add a check to ensure item.timetable is not undefined
			    if (item.timetable) {
			        const stop = item.timetable.find(tt => tt.stationTelecode === this.keyword);
			        return stop ? stop.stopTime : '-';
			    }
			    return '-'; // Return a default value
			}
		}
	}
</script>

<style>
	.page {
		min-height: 100vh;
	}

	.status-bar {
		height: var(--status-bar-height);
	}
</style>