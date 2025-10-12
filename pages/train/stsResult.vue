<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		<view class="ux-bg-primary" style="height:  var(--status-bar-height);">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">车次查询</text>
			<br><br>
			<text class="ux-text-small ux-opacity-5">共查询到 {{this.showData.length}} 个车次</text>
			<view v-if="isVague" class="ux-text-small ux-opacity-5 ux-color-primary ux-mt-small">
				<text class="icon">&#xe88e;</text>&nbsp;当前查询结果包含同城所有车站间的列车
			</view>
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
								{{item.fromDepart}}
							</text>
							<br>
							<text class="ux-text-small">
								{{item.fromStationName || item.timetable[item.fromPos].station}}
							</text>
						</view>
						<view class="ux-text-center">
							<text class="consolas" style="font-size:40rpx;">
								{{item.numberKind + item.numberFull.join("/").replace(item.numberKind, "").replace(item.numberKind, "")}}
							</text>
							<view style="border-top: 0.1rpx solid #757575;width:30vw;margin: 5rpx 0;"></view>
							<text class="ux-text-small ux-opacity-5">
								{{item.passTime}}
							</text>
						</view>
						<view>
							<view class="ux-flex ux-align-items-start">
								<text class="consolas" style="font-size:45rpx;">
									{{item.toArrive}}
								</text>
								<text v-if="item.dayDiff!=0"
									style="font-size:20rpx">
									+{{item.dayDiff}}
								</text>
							</view>
							<text class="ux-text-small">
								{{item.toStationName || item.timetable[item.toPos].station}}
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
						<text class="ux-text-small">按历时</text>
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
	import {
		uniGet
	} from "@/scripts/req";

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
				"filterTypeState": "GDCZTK12345678SLY",
				"isVague": false, // 新增：是否为同城查询
				"fromCity": "", // 新增：出发城市
				"toCity": "" // 新增：到达城市
			}
		},
		async onLoad(options) {
			this.from = options.from;
			this.to = options.to;
			this.error = "";
			this.date = options.date;
			// 检查 city 参数
			this.isVague = options.city === 'true';

			const c = uni.getStorageSync("search");
			uni.setStorage({
				key: 'search',
				data: c + 1
			});

			// 在查询前获取城市信息（用于本地查询）
			try {
				let selectionA = uni.getStorageSync("train_sts_fieldA");
				let selectionB = uni.getStorageSync("train_sts_fieldB");
				this.fromCity = selectionA ? selectionA.city : '';
				this.toCity = selectionB ? selectionB.city : '';
			} catch (e) {
				console.error("获取城市信息失败", e);
			}

			try {
				await this.fillInData();
			} catch (error) {
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

						// 调整 API URL：如果开启同城查询，则添加 &city=true 参数
						let url = `https://data.railgo.zenglingkun.cn/api/train/sts_query?from=${this.from}&to=${this.to}&date=${this.date}`;
						if (this.isVague) {
							url += "&city=true";
						}

						const resp = await uniGet(url);
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

						// API 模式下，API 已经计算好 fromDepart/toArrive/dayDiff/passTime
						// 并且已经按出发时间排序
						this.data = result.map(item => {
							// 提取实际匹配的出发/到达站名称
							const fromStnInfo = item.timetable.find(stop => stop.stationTelecode === item.fromStationTelecode);
							const toStnInfo = item.timetable.find(stop => stop.stationTelecode === item.toStationTelecode);
							
							return {
								...item,
								// 将 API 提供的关键信息映射到现有结构
								fromDepart: item.fromDepart,
								toArrive: item.toArrive,
								dayDiff: item.dayDiff,
								passTime: item.passTime,
								fromStationName: fromStnInfo ? fromStnInfo.station : '',
								toStationName: toStnInfo ? toStnInfo.station : ''
							}
						});
						this.showData = this.data;

						// 再次调用排序，确保按用户默认设置排序 (API 默认按出发时间，但这里确保遵循组件的默认/上次设置)
						this.radioSortChange({
							detail: {
								value: this.sortState || "departure"
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

						let fromTelecodes = [this.from];
						let toTelecodes = [this.to];

						if (this.isVague && this.fromCity && this.toCity && this.fromCity !== this.toCity) {
							// 步骤1: 查询出发城市和到达城市的所有车站 telecode
							const fromStns = toRaw(await doQuery(
								`SELECT telecode, trainList, name FROM stations WHERE city='${this.fromCity}' AND trainList IS NOT NULL`,
								["telecode", "trainList", "name"]
							));
							const toStns = toRaw(await doQuery(
								`SELECT telecode, trainList, name FROM stations WHERE city='${this.toCity}' AND trainList IS NOT NULL`,
								["telecode", "trainList", "name"]
							));
							
							fromTelecodes = fromStns.map(stn => stn.telecode);
							toTelecodes = toStns.map(stn => stn.telecode);
							
							// 合并所有 trainList 以找出公共列车
							let allFromTrainList = [];
							fromStns.forEach(stn => allFromTrainList.push(...stn.trainList));
							let allToTrainList = [];
							toStns.forEach(stn => allToTrainList.push(...stn.trainList));
							
							// 找出公共列车号
							let commonTrainSet = new Set(allFromTrainList.filter(train => allToTrainList.includes(train)));
							var commonTrainList = Array.from(commonTrainSet);
							
						} else {
							// 点对点查询（原逻辑）
							let fromStn = toRaw(await doQuery("SELECT trainList FROM stations WHERE telecode='" + this.from +
								"'", ["trainList"]))[0];
							let toStn = toRaw(await doQuery("SELECT trainList FROM stations WHERE telecode='" + this.to + "'",
								["trainList"]))[0];
								
							if (!fromStn || !toStn || !fromStn.trainList || fromStn.trainList.length == 0 || !toStn.trainList || toStn.trainList.length == 0) {
								uni.hideLoading();
								this.$refs.error_noky.open();
								return;
							}
							
							var commonTrainList = fromStn.trainList.filter((i) => toStn.trainList.includes(i));
						}
						
						if (commonTrainList.length === 0) {
							uni.hideLoading();
							this.$refs.error_nosuch.open();
							return;
						}

						// 步骤2: 查询所有公共列车的详情
						let all = toRaw(await doQuery(
							"SELECT code, number, numberFull, numberKind, timetable, rundays FROM trains WHERE number IN ('" +
							commonTrainList.join("','") + "')", ["code", "number", "numberFull", "numberKind", "timetable",
								"rundays"
							]));
							
						// 步骤3: 遍历列车，匹配最早出发和最晚到达的车站
						let results = [];
						all.forEach((k) => {
							let fromPos = -1;
							let toPos = -1;
							let fromStop = null;
							let toStop = null;
							
							if (!k.rundays.includes(this.date)) {
								return; // 跳过日期不匹配的列车
							}
							
							// 查找最早的出发站
							for (var i = 0; i < k.timetable.length; i++) {
								if (fromTelecodes.includes(k.timetable[i].stationTelecode)) {
									fromPos = i;
									fromStop = k.timetable[i];
									break;
								}
							}
							
							// 从出发站之后查找第一个在 toTelecodes 列表中的车站
							if (fromPos != -1) {
								for (var i = fromPos + 1; i < k.timetable.length; i++) {
									if (toTelecodes.includes(k.timetable[i].stationTelecode)) {
										toPos = i;
										toStop = k.timetable[i];
										// 找到即可停止，保证顺序
										break;
									}
								}
							}
							
							if (fromStop && toStop && fromPos < toPos) {
								k.fromPos = fromPos;
								k.toPos = toPos;
								k.showFlag = true;
								k.fromDepart = fromStop.depart;
								k.toArrive = toStop.arrive;
								k.dayDiff = toStop.day - fromStop.day;
								k.passTime = this.calculateTimeDifference(
									k.fromDepart, 
									k.toArrive, 
									k.dayDiff
								);
								// 添加实际匹配的站名，用于显示
								k.fromStationName = fromStop.station;
								k.toStationName = toStop.station;
								results.push(k);
							}
						});
			
						this.data = results;
						this.showData = this.data;
						
						// 默认按出发时间排序 (Local 模式下需要手动排序)
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
				// 如果是当天，且 endTime < startTime，通常意味着是次日到达，但在计算中 daysLater 应该已经处理了。
				// 再次检查确保非负
				if (differenceMinutes < 0) differenceMinutes += (daysLater > 0 ? 0 : 24 * 60); 
				
				const hours = Math.floor(differenceMinutes / 60);
				const minutes = differenceMinutes % 60;
				// 适配网络模式的格式
				return `${hours}时${minutes.toString().padStart(2, '0')}分`;
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
						// 按历时排序
						this.showData = this.data.sort((a, b) => {
							// 将 'X时Y分' 转换为总分钟数进行比较
							const parseDuration = (passTime) => {
								if (!passTime) return 999999;
								const match = passTime.match(/(\d+)时(\d+)分/);
								if (match) {
									return parseInt(match[1]) * 60 + parseInt(match[2]);
								}
								return 999999;
							}
							return parseDuration(a.passTime) - parseDuration(b.passTime);
						});
						break;

					case "departure":
						this.showData = this.data.sort((a, b) => {
							// 按出发时间排序 (HH:MM 字符串比较)
							const departA = a.fromDepart || "99:99";
							const departB = b.fromDepart || "99:99";
							if (departA > departB) {
								return 1;
							}
							if (departA < departB) {
								return -1;
							}
							return 0;
						});
						break;

					case "arrival":
						this.showData = this.data.sort((a, b) => {
							// 确保使用 toArrive 和 dayDiff 字段
							const arriveTimeA = (a.dayDiff * 1440) + this.timeToMinutes(a.toArrive);
							const arriveTimeB = (b.dayDiff * 1440) + this.timeToMinutes(b.toArrive);
							
							return arriveTimeA - arriveTimeB;
						});
						break;

					default:
						console.log("WHAT THE FUCK R U DOING?");
				}
				this.$refs.menu_sort.close();
			},
			timeToMinutes(timeStr) {
				if (!timeStr) return 0;
				const [hours, minutes] = timeStr.split(':').map(Number);
				return hours * 60 + minutes;
			},
			radioFilterChange: function(e) {
				this.filterTypeState = e.detail.value.join("");
				this.showData = this.data.filter((i) => {
					const firstChar = i.number.charAt(0);
					if (!isNaN(parseInt(firstChar))) {
						return e.detail.value.includes("12345678");
					}
					return e.detail.value.join("").includes(firstChar);
				});
			}
		}
	}
</script>