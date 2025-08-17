<template>
	<view class="ux-bg-grey5 page">
		<!-- headers begin -->
		<view class="ux-bg-primary status-bar"></view>
		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
		</view>
		<!-- headers end -->
		<view class="ux-padding">
			<!-- 站名小卡片 -->
			<view class="ux-bg-white ux-border-radius">
				<view class="ux-pl ux-pr ux-pb-small ux-pt ux-flex ux-align-items-center ux-justify-content-center">
					<view class="ux-flex ux-align-items-center">
						<view v-for="(tag,i) in this.data.type" :key="i">
							<text class="ux-badge ux-color-white ux-mr-small"
								:style="'background-color:'+this.badgeFlag[tag]">{{tag}}</text>
						</view>
					</view>
					<view>
						<text class="ux-h3">{{this.data.name}}站</text><br>
						<text class="ux-text-small">{{this.data.pinyin}} Station</text>
					</view>
				</view>
				<view class="ux-flex ux-space-between ux-color-white ux-pt-small ux-pb-small ux-pl ux-pr ux-text-small"
					style="border-bottom-left-radius:10rpx;border-bottom-right-radius:10rpx;"
					:style="'background-color:'+(this.data.type.includes('客')?'#114598':'#eeba67')">
					<text>{{this.data.pinyinTriple}}/-{{this.data.telecode}}</text>
					<text>{{this.data.bureau}} {{this.data.belong}}辖</text>
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
				<view v-if="this.trains.length!=0">
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
						<text class="ux-text-small ux-opacity-5">共查询到 {{this.showTrains.length}} 个车次</text>
						<br>
					</view>
					<navigator v-for="(item,index) in showTrains" :key="index"
						:url="'/pages/train/trainResult?keyword='+item.code+'&date='+getToday()">
						<view class="ux-bg-white ux-border-radius ux-mt-small ux-flex">
							<view style="border-bottom-left-radius: 10rpx; border-top-left-radius:10rpx;"
								:style="'background-color:'+colorMap[item.number[0]]">
								&nbsp;&nbsp;
							</view>
							<view class="ux-flex ux-align-items-center ux-space-between ux-pr ux-pt ux-pb ux-pl-small"
								style="width:100%;">
								<view style="width:calc(100% - 60rpx);">
									<view class="ux-flex ux-space-between">
										<!--
										<view>
											<text class="consolas"
												style="font-size:40rpx;">{{item.numberKind}}{{item.numberFull.join("/").replaceAll(item.numberKind, "")}}</text>
											<br>
											<text class="ux-text-small">{{item.timetable[0].station}} ⋙
												{{item.timetable[item.timetable.length-1].station}}</text>
										</view>
										-->
										<view class="ux-flex" style="padding-top:8rpx;">
											<view class="ux-text-center ux-mr-small" style="min-width:80rpx">
												<text class="ux-text"
													v-if="item.timetable[0].stationTelecode != this.keyword">{{item.timetable[item.indexStopThere].arrive}}</text>
												<text class="ux-text"
													v-if="item.timetable[0].stationTelecode == this.keyword">--:--</text>
												<br>
												<text class="ux-text-small ux-opacity-5">到达</text>
											</view>
											<view class="ux-text-center ux-mr-small" style="min-width:80rpx">
												<text class="ux-text"
													v-if="item.timetable[item.timetable.length - 1].stationTelecode != this.keyword">{{item.timetable[item.indexStopThere].depart}}</text>
												<text class="ux-text"
													v-if="item.timetable[item.timetable.length - 1].stationTelecode == this.keyword">--:--</text>
												<br>
												<text class="ux-text-small ux-opacity-5">出发</text>
											</view>
											<view class="ux-text-center">
												<text
													class="ux-text">{{item.timetable[item.indexStopThere].stopTime}}'</text>
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
				<view v-if="this.trains.length==0">
					<view class="ux-padding ux-text-center">本站不办理客运业务或无列车停靠</view>
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
						:checked="this.sortState=='departure'">
						<text class="ux-text-small">按发车时间</text>
					</radio>
					<br>
					<radio color="#114598" value="arrival" class="ux-mr ux-mt-small"
						:checked="this.sortState=='arrival'">
						<text class="ux-text-small">按到达时间</text>
					</radio>
					<br>
					<radio color="#114598" value="stop" class="ux-mr ux-mt-small" :checked="this.sortState=='stop'">
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
				<uv-divider></uv-divider>
				<text class="ux-h4">到发筛选</text>
				<br>
				<checkbox-group class="ux-mt-small" @change="radioSourceChange">
					<view class="ux-flex ux-space-between">
						<checkbox color="#114598" value="P" class="ux-mr ux-mt-small"
							:checked="this.filterSourceState.includes('P')">
							<text class="ux-text-small">过路</text>
						</checkbox>
						<checkbox color="#114598" value="D" class="ux-mr ux-mt-small"
							:checked="this.filterSourceState.includes('D')">
							<text class="ux-text-small">本站始发</text>
						</checkbox>
						<checkbox color="#114598" value="A" class="ux-mr ux-mt-small"
							:checked="this.filterSourceState.includes('A')">
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
			this.fillInData();
		},
		methods: {
			// 返回上一页
			back: function() {
				uni.navigateBack();
			},
			fillInData: async function() {
				this.data = toRaw(await doQuery("SELECT * FROM stations WHERE telecode='" + this.keyword + "'",
					KEYS_STRUCT_STATIONS))[0];
				if (this.data.trainList.length == 0) {
					return;
				}
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
				}, this);
				this.radioSortChange({
					detail: {
						value: "departure"
					}
				});
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
							if (a.timetable[a.indexStopThere].stopTime > b.timetable[b.indexStopThere]
								.stopTime) {
								return 1;
							}
							if (a.timetable[a.indexStopThere].stopTime < b.timetable[b.indexStopThere]
								.stopTime) {
								return -1;
							}
							return 0;
						});
						break;

					case "departure":
						this.showTrains = this.trains.sort((a, b) => {
							if (a.timetable[a.indexStopThere].depart > b.timetable[b.indexStopThere]
								.depart) {
								return 1;
							}
							if (a.timetable[a.indexStopThere].depart < b.timetable[b.indexStopThere]
								.depart) {
								return -1;
							}
							return 0;
						});
						break;

					case "arrival":
						this.showTrains = this.trains.sort((a, b) => {
							if (a.timetable[a.indexStopThere].arrive > b.timetable[b.indexStopThere].arrive) {
								return 1;
							}
							if (a.timetable[a.indexStopThere].arrive < b.timetable[b.indexStopThere].arrive) {
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
				this.showTrains = this.trains.filter((i) => {
					return e.detail.value.join("").includes(i.number.charAt(0));
				});
			},
			radioSourceChange: function(e) {
				this.filterSourceState = e.detail.value.join("");
				this.showTrains = this.trains.filter((i) => {
					return (
						(i.timetable[0].stationTelecode != this.keyword && i.timetable[i.timetable.length -
							1].stationTelecode != this.keyword && this.filterSourceState.includes("P")) ||
						(i.timetable[0].stationTelecode == this.keyword && this.filterSourceState.includes(
							"D")) ||
						(i.timetable[i.timetable.length - 1].stationTelecode == this.keyword && this
							.filterSourceState.includes("A"))
					);
				});
			},
			getToday: function() {
				const date = new Date();
				return `${date.getFullYear()}${('0' + (date.getMonth() + 1)).slice(-2)}${('0' + date.getDate()).slice(-2)}`;
			}
		}
	}
</script>