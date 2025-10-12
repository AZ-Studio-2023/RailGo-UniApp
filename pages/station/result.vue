<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		<view class="ux-bg-primary" style="height: var(--status-bar-height);">&nbsp;</view>

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
						<text v-if="data.province || (data.level && data.level !== '未知')" class="ux-text-small ux-opacity-7">
							<template v-if="data.province">
								{{ data.province === data.city ? data.province : (data.province + (data.city || '')) }}
							</template>
							
							<template v-if="data.province && data.level && data.level !== '未知'"> | </template>
							
							<template v-if="data.level && data.level !== '未知'">{{data.level}}</template>
							<br>
						</text>
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
            <view class="ux-text-center ux-padding-small ux-mb ux-h6"  v-if="topTabList[selectIndex] && topTabList[selectIndex].name === '大屏'"
				style="background-color:#e9eef5;border:1px solid #114598;border-radius:10rpx;color:#114598; margin-top: 20rpx">
				<text class="ux-bold">信息仅供参考 请以车站现场公告为准</text>
			</view>
			<view class="ux-pt dark-table-wrapper" v-if="topTabList[selectIndex] && topTabList[selectIndex].name === '大屏'" style="margin-top: 20rpx;">
				<uni-table :loading="bigScreenLoading" emptyText="暂无数据" class="dark-table">
					<uni-tr>
						<uni-th align="center">车次</uni-th>
						<uni-th align="center">状态</uni-th>
						<uni-th align="center">始发站</uni-th>
						<uni-th align="center">终到站</uni-th>
						<uni-th align="center">开点</uni-th>
						<uni-th align="center">候车/检票</uni-th>
					</uni-tr>
					<uni-tr v-for="(row, index) in bigScreenData" :key="index">
						<uni-td align="center">{{ row[0] }}</uni-td>
						<uni-td align="center">
							<text :style="{color: getStatusColor(row[5])}">{{ row[5] }}</text>
						</uni-td>
						<uni-td align="center">{{ row[1] }}</uni-td>
						<uni-td align="center">{{ row[2] }}</uni-td>
						<uni-td align="center">{{ formatDepartureTime(row[3]) }}</uni-td>
						<uni-td align="center">{{ row[4] }}</uni-td>
					</uni-tr>
				</uni-table>
				
				<view v-if="bigScreenData.length==0 && !bigScreenLoading" class="ux-padding ux-text-center">
					<text style="color: white; opacity: 0.6;">本站暂无大屏数据</text>
				</view>

			</view>
			<view class="ux-padding ux-text-center" v-if="topTabList[selectIndex] && topTabList[selectIndex].name === '路线'">
				<text>暂未开放，敬请期待</text>
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
							:checked="filterTypeState.includes('12345678')">
							<text class="ux-text-small">普客</text>
						</checkbox>
						<checkbox color="#114598" value="S" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('S')">
							<text class="ux-text-small">市域</text>
						</checkbox>
						<checkbox color="#114598" value="LY" class="ux-mr ux-mt-small"
							:checked="filterTypeState.includes('LY')">
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
    </view>
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
	import {uniGet} from "../../scripts/req";

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
				// 默认只包含“车次”和“路线”，如果为客运站则在 fillInData 中添加“大屏”
				topTabList: [{
					name: '车次',
				}, {
					name: '路线'
				}],
				selectIndex: 0,
				trains: [],
				showTrains: [],
				colorMap: TRAIN_KIND_COLOR_MAP,
				filterTypeState: ["G", "D", "C", "Z", "T", "K", "12345678", "S", "LY"],
				filterSourceState: ["P", "D", "A"],
				sortState: "departure",
				// --- 新增大屏数据和加载状态 ---
				bigScreenData: [],
				bigScreenLoading: false
				// -------------------------------
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

				let success = false;
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
							uni.redirectTo({
								url: '/pages/404/404'
							});
							return;
						}
						this.data = result.data || {};
						this.trains = result.trains || [];
						success = true;
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
						const stationData = (await doQuery("SELECT * FROM stations WHERE telecode='" + this.keyword + "'", KEYS_STRUCT_STATIONS))[0];
						this.data = toRaw(stationData);
						
						if (this.data.trainList && this.data.trainList.length > 0) {
							// 获取所有列车数据
							const rawTrains = await doQuery(
								"SELECT code, number, numberFull, numberKind, timetable FROM trains WHERE number IN ('" +
								this.data.trainList.join("','") + "')", ["code", "number", "numberFull", "numberKind",
									"timetable"
								]);
							
							// 遍历并处理每个列车对象，添加 arrive, depart, stopTime
							this.trains = rawTrains.map(item => {
								const stop = item.timetable.find(tt => tt.stationTelecode === this.keyword);
								if (stop) {
									return {
										...item,
										arrive: stop.arrive,
										depart: stop.depart,
										stopTime: stop.stopTime,
										fromStation: item.timetable[0],
										toStation: item.timetable[item.timetable.length - 1],
										indexStopThere: item.timetable.findIndex(tt => tt.stationTelecode === this.keyword)
									};
								}
								return item; // 如果没有找到匹配的站点，返回原始对象
							});

						} else {
							this.trains = [];
						}
						success = !!this.data.name; // 检查是否成功获取到车站数据
					} catch (error) {
						console.error("本地数据加载失败", error);
					}
				}
				uni.hideLoading();
				if (success) {
					// 1. 检查是否为客运站
					const isPassengerStation = Array.isArray(this.data.type) && this.data.type.includes("客");
                    // IMPORTANT: Reset topTabList to default before potentially adding '大屏'
                    this.topTabList = [{ name: '车次' }, { name: '路线' }];
                    
					if (isPassengerStation) {
						// 2. 如果是客运站，添加“大屏”tab到索引1（“车次”和“路线”之间）
						this.topTabList.splice(1, 0, { name: '大屏' });
						uni.showLoading({
							title: '加载大屏数据'
						})
						// 3. 异步获取大屏数据
						await this.getBigScreenData();
						uni.hideLoading()
					}
					
					this.applySortingAndFiltering(); // 初始加载时应用一次排序和筛选
				}

				
			},
			tabChange: function(e) {
				this.selectIndex = e.index;
			},
			// --- 获取大屏数据的方法 ---
			getBigScreenData: async function() {

				if (!this.data.name) return; // 车站名不存在则不查询
				this.bigScreenLoading = true;
				try {
					// 移除 '站' 字并进行 URI 编码
					const stationName = this.data.name.replace(/站$/, ''); 
					const encodedName = encodeURIComponent(stationName);
					const url = `https://screen.data.railgo.zenglingkun.cn/station/${encodedName}`;
					const resp = await uniGet(url);
					
					// 检查响应数据结构
					if (resp.data && Array.isArray(resp.data.data)) {
						this.bigScreenData = resp.data.data;
					} else {
						this.bigScreenData = [];
					}
				} catch (error) {
					console.error("大屏数据加载失败", error);
					// 不弹出 toast，静默失败，因为主数据已加载
					this.bigScreenData = [];
				} finally {
					this.bigScreenLoading = false;
				}
			},
			
			// --- 时间格式化方法 ---
			formatDepartureTime(fullTime) {
				if (!fullTime) return '--:--';
				// 提取时间部分 (HH:mm)
				const parts = fullTime.split(' ');
				return parts.length > 1 ? parts[1].substring(0, 5) : '--:--';
			},
			
			// --- 状态颜色逻辑 ---
			getStatusColor(status) {
				if (!status) return 'white'; // 默认白色
				if (status.includes('正在检票')) {
					return '#27ae60'; // 绿色
				} else if (status.includes('停止检票') || status.includes('晚点')) {
					return '#e74c3c'; // 红色
				} else if (status.includes('早点')) {
					return '#27ae60'; // 绿色
				}
				return 'white'; // 默认白色
			},
			// -------------------------------
			openSortMenu: function() {
				this.$refs.menu_sort.open();
			},
			openFilterMenu: function() {
				this.$refs.menu_filter.open();
			},
			
			// 新增的集中处理排序和筛选的函数
			applySortingAndFiltering: function() {
				let filteredTrains = [...this.trains];

				// 1. 先进行筛选
				filteredTrains = filteredTrains.filter(i => {
					// 筛选车种
					const firstChar = i.number.charAt(0);
					let typeMatch = false;
					if (firstChar >= '1' && firstChar <= '8') {
						typeMatch = this.filterTypeState.includes("12345678");
					} else if (firstChar === 'L' || firstChar === 'Y') {
						typeMatch = this.filterTypeState.includes("LY");
					} else {
						typeMatch = this.filterTypeState.includes(firstChar);
					}
					
					// 筛选到发
					const fromCode = (i.fromStation && i.fromStation.stationTelecode) || (i.timetable && i.timetable[0] && i.timetable[0].stationTelecode) || '';
					const toCode = (i.toStation && i.toStation.stationTelecode) || (i.timetable && i.timetable[i.timetable.length - 1] && i.timetable[i.timetable.length - 1].stationTelecode) || '';
					
					const sourceMatch = (
						(fromCode !== this.keyword && toCode !== this.keyword && this.filterSourceState.includes("P")) ||
						(fromCode === this.keyword && this.filterSourceState.includes("D")) ||
						(toCode === this.keyword && this.filterSourceState.includes("A"))
					);
					
					return typeMatch && sourceMatch;
				});

				// 2. 然后进行排序
				switch (this.sortState) {
					case "stop":
						filteredTrains.sort((a, b) => {
							const stopStrA = this.getStopTime(a);
							const stopStrB = this.getStopTime(b);
							
							// 将 '-' 视为最大值 (Infinity)，确保无停车时长的排在最后
							const stopTimeA = stopStrA === '-' ? Infinity : parseInt(stopStrA) || 0;
							const stopTimeB = stopStrB === '-' ? Infinity : parseInt(stopStrB) || 0;
							
							return stopTimeA - stopTimeB;
						});
						break;
					case "departure":
						filteredTrains.sort((a, b) => {
							const departA = this.getDepartTime(a);
							const departB = this.getDepartTime(b);

							// 将 '--:--' 视为最大值 '99:99'，确保无出发时间的排在最后
							const timeA = departA === '--:--' ? '99:99' : departA;
							const timeB = departB === '--:--' ? '99:99' : departB;
							
							return timeA.localeCompare(timeB);
						});
						break;
					case "arrival":
						filteredTrains.sort((a, b) => {
							const arriveA = this.getArriveTime(a);
							const arriveB = this.getArriveTime(b);
							
							// 将 '--:--' 视为最大值 '99:99'，确保无到达时间的排在最后
							const timeA = arriveA === '--:--' ? '99:99' : arriveA;
							const timeB = arriveB === '--:--' ? '99:99' : arriveB;
							
							return timeA.localeCompare(timeB);
						});
						break;
				}

				// 3. 最后更新展示列表
				this.showTrains = filteredTrains;
			},
			
			// 修复了这三个方法，确保状态正确更新为数组，并调用 applySortingAndFiltering
			radioSortChange: function(e) {
			    this.sortState = e.detail.value;
			    this.applySortingAndFiltering();
			    this.$refs.menu_sort.close();
			},
			radioFilterChange: function(e) {
				this.filterTypeState = e.detail.value;
			    this.applySortingAndFiltering();
			},
			radioSourceChange: function(e) {
				this.filterSourceState = e.detail.value;
			    this.applySortingAndFiltering();
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
			    if (item.timetable) {
			        const stop = item.timetable.find(tt => tt.stationTelecode === this.keyword);
			        return stop ? stop.arrive : '--:--';
			    }
			    return '--:--';
			},
			
			getDepartTime(item) {
			    if (item.depart) {
			        return item.depart;
			    }
			    if (item.timetable) {
			        const stop = item.timetable.find(tt => tt.stationTelecode === this.keyword);
			        return stop ? stop.depart : '--:--';
			    }
			    return '--:--';
			},
			
			getStopTime(item) {
			    if (item.stopTime) {
			        return item.stopTime;
			    }
			    if (item.timetable) {
			        const stop = item.timetable.find(tt => tt.stationTelecode === this.keyword);
			        return stop ? stop.stopTime : '-';
			    }
			    return '-';
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
	
	.dark-table-wrapper {
		background-color: #2c3e50; 
		border-radius: 10rpx;
		padding: 10rpx;
	}

	.dark-table .uni-table-th,
	.dark-table .uni-table-td {
		color: white !important;
		background-color: #3b506b !important; 
		border-color: #3b506b !important; 
	}

	.dark-table .uni-table-th {
		background-color: #1f3041 !important;
	}
</style>