<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		<view class="ux-bg-primary" style="height: var(--status-bar-height);">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
		</view>
		<view class="ux-pl ux-pr ux-pb">

			<view class="ux-bg-white ux-border-radius">
				<view class="ux-flex ux-space-between ux-pt ux-pl ux-pr ux-align-items-center">
					<view>
						<text class="ux-bold consolas" style="font-size:60rpx;"
							:style="'color:'+cardColor">{{carData.numberKind || ''}}</text>
						<text class="consolas"
							style="font-size:50rpx;padding-left:5rpx;">{{(carData.numberFull || []).join("/").replace(carData.numberKind, "").replace(carData.numberKind, "")}}</text>
					</view>
					<text class="ux-badge ux-text-small ux-color-white" style="padding:5rpx 15rpx;"
						:style="'background-color:'+cardColor">{{carData.type || ''}}</text>
				</view>
				<view class="ux-flex ux-space-between ux-mt-small ux-pl ux-pr ux-pt-small ux-color-white"
					:style="'background-color:'+cardColor">
					<text
						class="ux-text-small">{{(carData.timetable && carData.timetable[0] ? carData.timetable[0].station : '')}}
						⋙
						{{(carData.timetable && carData.timetable.length > 0 ? carData.timetable[carData.timetable.length-1].station : '')}}</text>
					<text class="ux-text-small">{{carData.bureauName || ''}}{{carData.runner || ''}}&nbsp;&nbsp;</text>
				</view>
				<view class="ux-pb-small" style="border-bottom-left-radius:10rpx; border-bottom-right-radius:10rpx;"
					:style="'background-color:'+cardColor">
				</view>
			</view>
			<br>
			<view class="ux-padding-small ux-h6 ux-text-center"
				style="background-color:#e9eef5;border:1px solid #114598;border-radius:10rpx;color:#114598;">
				<text class="ux-bold">信息仅供参考 请以铁路运营企业实际运用为准</text>
			</view>
			<view v-if="carData.rundays && !carData.rundays.includes(date)">
				<view class="ux-padding-small ux-h6 ux-text-center ux-mt-small"
					style="background-color:#f8eceb;border:1px solid #c0392b;border-radius:10rpx;color:#c0392b;">
					<text class="ux-bold">此车次在选定日期计划不开行 请注意核对信息</text>
				</view>
			</view>
			<view class="ux-flex ux-justify-content-center">
				<uv-tabs :list="topTabList" lineWidth="60" lineColor="#114599" :activeStyle="{
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
				<uni-section title="时刻表" type="line" style="background-color: transparent;"
					title-font-size="28rpx"></uni-section>
				<uni-table style="border:none">
					<uni-tr style="border:none">
						<uni-th style="border:none" width="10">
							<image style="width:1px;height:1px;transform:translateY(10px) scale(40);" mode="aspectFit"
								src="@/static/station-mark-th.png"></image>
						</uni-th>
						<uni-th>车站名</uni-th>
						<uni-th width="70">车次</uni-th>
						<uni-th width="60">到达</uni-th>
						<uni-th width="60">出发</uni-th>
						<uni-th width="50">停车</uni-th>
						<uni-th width="80">里程</uni-th>
						<uni-th width="100">区间均速</uni-th>
						<uni-th width="60">日期</uni-th>
					</uni-tr>
					<uni-tr v-for="(item,index) in (carData.timetable || [])" :key="index" style="border:none"
						hover-class="ux-bg-grey5">
						<uni-td style="border:none">
							<image style="width:1px;height:1px;transform:translateY(-5px) scale(40)" mode="aspectFit"
								src="@/static/station-mark-stop.png"></image>
						</uni-td>
						<uni-td style="border:none">
							<navigator
								:url="'/pages/station/result?keyword='+(item.stationTelecode || '')+'&vague=false'">
								{{item.station || ''}}
							</navigator>
						</uni-td>
						<uni-td style="border:none">{{item.trainCode || ''}}</uni-td>
						<uni-td style="border:none">{{item.arrive || ''}}</uni-td>
						<uni-td style="border:none">{{item.depart || ''}}</uni-td>
						<uni-td style="border:none">{{item.stopTime || '-'}}'</uni-td>
						<uni-td style="border:none">{{item.distance || '0'}} km</uni-td>
						<uni-td style="border:none">{{item.speed ? item.speed.toFixed(1) : '-'}} km/h</uni-td>
						<uni-td style="border:none">{{item.day || '当'}}日</uni-td>
					</uni-tr>
				</uni-table>

				<uni-section title="正晚点" type="line" style="background-color: transparent;"
					title-font-size="28rpx"></uni-section>
				<uni-table style="border:none">
					<uni-tr style="border:none">
						<uni-th style="border:none" align="center">车站</uni-th>
						<uni-th style="border:none" align="center">车次</uni-th>
						<uni-th style="border:none" align="center">预计到达时间<br>预计发车时间</uni-th>
						<uni-th style="border:none" align="center">实际到达时间<br>实际发车时间</uni-th>
						<uni-th style="border:none" align="center">停站时间</uni-th>
						<uni-th style="border:none" align="center">日期</uni-th>
						<uni-th style="border:none" align="center">状态</uni-th>
					</uni-tr>
					<uni-tr v-for="(item,index) in delay" :key="index" style="border:none" hover-class="ux-bg-grey5">
						<uni-td style="border:none" align="center">
							<view class="ux-flex ux-align-items-center ux-justify-content-center">
								<text>{{item.stationName || ''}}</text>
								<uni-badge v-if="getDelayBadgeText(item.delayMinutes, item.status).text"
									:text="getDelayBadgeText(item.delayMinutes, item.status).text"
									:type="getDelayBadgeText(item.delayMinutes, item.status).type"
									size="small"
									style="margin-left: 5px;"
								></uni-badge>
							</view>
						</uni-td>
						<uni-td style="border:none" align="center">{{item.trainNumber || ''}}</uni-td>
						<uni-td style="border:none" align="center">{{item.arrivalTime || '-'}}<br>{{item.departureTime || '-'}}</uni-td>
						<uni-td style="border:none" align="center">
							{{calculateActualTime(item.arrivalTime, item.delayMinutes, item.status)}}
							<br>
							{{calculateActualTime(item.departureTime, item.delayMinutes, item.status)}}
						</uni-td>
						<uni-td style="border:none" align="center">{{item.stopMinutes || ''}}'</uni-td>
						<uni-td style="border:none" align="center">{{item.arrivalDate || ''}}</uni-td>
						<uni-td style="border:none" align="center" :style="getDelayStatusColor(item.delayMinutes, item.status)">
							{{formatDelayStatus(item.delayMinutes, item.status)}}
						</uni-td>
					</uni-tr>
					<uni-tr v-if="delay.length === 0" style="border:none">
						<uni-td style="border:none" colspan="7" align="center"  v-if="isOnlyOfflineMode == false">
							暂无正晚点信息或加载失败
						</uni-td>
						<uni-td style="border:none" colspan="7" align="center" class="ux-color-gray" v-if="isOnlyOfflineMode">
							仅离线模式下无法使用该功能
						</uni-td>
					</uni-tr>
				</uni-table>
			</view>

			<view v-if="selectIndex==1">
				<uni-section title="担当" type="line" style="background-color: transparent;"
					title-font-size="28rpx"></uni-section>
				<view class="ux-bg-white ux-border-radius ux-padding"
					v-if="(carData.carOwner || '')+(carData.runner || '')+(carData.car || '')!=''">
					<view class="ux-flex ux-space-between">
						<view class="ux-flex ux-align-items-center" v-if="carData.carOwner">
							<view class="ux-pt-small">
								<text class="ux-color-primary icon" style="font-size:60rpx;">&#xe7fd;</text>
							</view>
							<view class="ux-pl-small">
								<text class="ux-text-small ux-opacity-5">
									值乘单位
								</text>
								<br>
								<text>
									{{carData.runner || ''}}
								</text>
							</view>
						</view>
						<view class="ux-flex ux-align-items-center" v-if="carData.carOwner">
							<view class="ux-pt-small">
								<text class="ux-color-primary icon" style="font-size:60rpx;">&#xe570;</text>
							</view>
							<view class="ux-pl-small">
								<text class="ux-text-small ux-opacity-5">
									车辆归属
								</text>
								<br>
								<text>
									{{carData.carOwner || ''}}
								</text>
							</view>
						</view>
					</view>
					<uv-divider v-if="(carData.carOwner || '')+(carData.runner || '')!=''" />
					<view v-if="carData.car">
						<view class="ux-flex ux-space-between">
							<text class="ux-text-small ux-opacity-5">车型</text>
							<text>{{carData.car || ''}}</text>
						</view>
						<view class="ux-flex ux-space-between ux-align-items-center">
							<view>
								<view class="ux-pr-small ux-flex ux-align-items-center ux-mt-small">
									<text class="ux-color-primary icon" style="font-size:50rpx;">&#xe642;</text>
									<view class="ux-pl-small ux-text-small">
										<text>{{carMap[carData.car.replace(" 重联","")] ? carMap[carData.car.replace(" 重联","")][0] : ''}}节编组</text>
									</view>
								</view>
								<view class="ux-pr-small ux-flex ux-align-items-center ux-mt-small">
									<text class="ux-color-primary icon" style="font-size:50rpx;">&#xe569;</text>
									<view class="ux-pl-small ux-text-small">
										<text>{{carMap[carData.car.replace(" 重联","")] ? carMap[carData.car.replace(" 重联","")][3] : ''}}km/h</text>
									</view>
								</view>
								<view class="ux-pr-small ux-flex ux-align-items-center ux-mt-small">
									<text class="ux-color-primary icon" style="font-size:50rpx;">&#xe5c3;</text>
									<view class="ux-pl-small ux-text-small">
										<text>{{carMap[carData.car.replace(" 重联","")] ? carMap[carData.car.replace(" 重联","")][1] : ''}}</text>
									</view>
								</view>
								<view class="ux-pr-small ux-flex ux-align-items-center ux-mt-small">
									<text class="ux-color-primary icon" style="font-size:50rpx;">&#xe556;</text>
									<view class="ux-pl-small ux-text-small">
										<text>{{carMap[carData.car.replace(" 重联","")] ? carMap[carData.car.replace(" 重联","")][2] : ''}}</text>
									</view>
								</view>
							</view>
							<view class="ux-mt-small">
								<image v-if="carMap[carData.car.replace(' 重联','')]"
									:src="carMap[carData.car.replace(' 重联','')][4]" mode="aspectFit"
									style="max-width:350rpx; height: 200rpx;overflow: hidden; border-radius: 500rpx;"></image>
							</view>
						</view>
						<view v-if="['G','D','C'].includes(carData.numberKind)">
							<br>
							<navigator :url="'/pages/emu/result?keyword='+train">
								<button class="ux-color-white ux-bg-primary" size="mini"
									style="margin:none;width:100%;">
									<view class="ux-flex ux-align-items-center ux-justify-content-center">
										<text class="icon">&#xe570;</text>
										&nbsp;查询具体担当信息
									</view>
								</button>
							</navigator>
						</view>
					</view>
				</view>
				<view v-if="(carData.carOwner || '')+(carData.runner || '')+(carData.car || '')==''"
					class="ux-padding ux-text-center">
					暂无担当
				</view>
				<uni-section title="交路" type="line" style="background-color: transparent;"
					title-font-size="28rpx"></uni-section>
				<navigator v-for="(item,index) in (carData.diagram || [])" :key="index"
					:url="'/pages/train/trainResult?keyword='+item.train_num+'&date='+date">
					<view class="ux-bg-white ux-border-radius ux-mt-small ux-flex">
						<view style="border-bottom-left-radius: 10rpx; border-top-left-radius:10rpx;"
							:style="'background-color:'+colorMap[item.train_num[0]]">
							&nbsp;&nbsp;
						</view>
						<view class="ux-flex ux-align-items-center ux-space-between ux-pr ux-pt ux-pb ux-pl-small"
							style="width:100%;">
							<view style="width:calc(100% - 70px);">
								<view class="ux-flex ux-align-items-center">
									<text class="consolas" style="font-size:40rpx;">{{item.train_num || ''}}</text>
								</view>
								<text class="ux-text-small">{{item.from ? item.from[0] : ''}}
									{{item.from ? item.from[1] : ''}} ⋙ {{item.to ? item.to[0] : ''}}
									{{item.to ? item.to[1] : ''}}</text>
							</view>
							<text class="ux-text"><text class="icon">&#xe5c8;</text></text>
						</view>
					</view>
				</navigator>
				<view v-if="(carData.diagram || []).length==0" class="ux-padding-large ux-text-center">
					暂无交路
				</view>
				<uni-section title="开行日" type="line" style="background-color: transparent;"
					title-font-size="28rpx"></uni-section>
				<calendar class="ux-bg-white ux-border-radius"
					:highlighted-dates="(()=>{var l=[]; (carData.rundays || []).forEach((i)=>{l.push({date: i.replace(/^(\d{4})(\d{2})(\d{2})$/, '$1-$2-$3'), bgColor: '#114598'});});return l;})()">
				</calendar>
				<view class="ux-flex ux-space-between ux-pt-small">
					<text class="ux-text-small ux-opacity-4">*实际开行请以车站公告为准</text>
					<view class="ux-flex ux-align-items-center ux-nowrap">
						<view class="ux-bg-primary ux-padding-small" style="width:0.2vh;height:0.2vh;"></view>
						<text>&nbsp;当日开行</text>
					</view>
				</view>
				
			</view>
			
			<view class="ux-padding ux-text-center" v-if="selectIndex==2">
				<text>暂未开放，敬请期待</text>
			</view>

			<br>
			<view class="ux-flex ux-row ux-justify-content-center">
				<text class="ux-text-small ux-opacity-4">—— · ——</text>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		queryMainKey
	} from "@/scripts/jsonDB.js";
	import {
		doQuery,
	} from "@/scripts/sqlite.js";
	import {
		KEYS_STRUCT_STATIONS,
		KEYS_STRUCT_TRAINS,
		TRAIN_KIND_COLOR_MAP,
		CAR_PERFORMANCE
	} from "@/scripts/config.js";
	import calendar from "@/components/diagram-calendar/diagram-calendar.vue";
	import {
		toRaw
	} from "@vue/reactivity";
	// 确保从 req.js 导入 uniGet 和 uniPost
	import {
		uniGet,
		uniPost
	} from "@/scripts/req";

	export default {
		components: {
			calendar
		},
		data() {
			return {
				"carData": {
					numberKind: '',
					numberFull: [],
					type: '',
					timetable: [],
					bureauName: '',
					runner: '',
					carOwner: '',
					car: '',
					rundays: [],
					diagram: [],
				},
				"colorMap": TRAIN_KIND_COLOR_MAP,
				"carMap": CAR_PERFORMANCE,
				"delay": [],
				"title": '',
				"date": "",
				"train": "",
				"keyword": "",
				"cardColor": "#114598",
				"topTabList": [{
					name: '时刻',
				}, {
					name: '运行',
				}, {
					name: '路径'
				}],
				"selectIndex": 0,
				"isOnlyOfflineMode": false // 新增：用于存储 ol 状态
			}
		},
		onLoad(options) {
			this.train = options.keyword ? options.keyword.split("/")[0] : '';
			this.keyword = options.keyword;
			this.title = this.train;
			this.date = options.date || '';

			const mode = uni.getStorageSync("mode");
			// 新增：检查 'ol' 状态
			this.isOnlyOfflineMode = uni.getStorageSync("ol") === true;

			const c = uni.getStorageSync("search");
			uni.setStorage({
				key: 'search',
				data: c + 1
			});
			this.fillInData(mode);
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
			/**
			 * 计算实际到达/发车时间 (HH:mm)
			 * @param {string|null} estimatedTime 预计时间，格式 "HH:mm"
			 * @param {number|null} delayMinutes 晚点分钟数 (负数为早点)
			 * @param {number|null} status 状态码 (1: 正点, 2: 晚点, 3: 早点)
			 * @returns {string} 实际时间或 '-'
			 */
			calculateActualTime: function(estimatedTime, delayMinutes, status) {
				// 1. 如果预计时间为 null/empty 或状态为 null/undefined，则为 "-"
				if (!estimatedTime || estimatedTime === '-' || status === null || status === undefined) {
					return '-';
				}

				// 2. 如果 delayMinutes 不是一个有效的数字，无法计算，返回 "-"
				if (typeof delayMinutes !== 'number' || isNaN(delayMinutes) || delayMinutes === null) {
					return '-';
				}

				// 3. 如果 status 为 1 且 delayMinutes 为 0，则为预计时间 (正点)
				if (status === 1 && delayMinutes === 0) {
					return estimatedTime;
				}

				// 时间计算逻辑: "HH:mm" + delayMinutes
				const parts = estimatedTime.split(':');
				let hours = parseInt(parts[0]);
				let minutes = parseInt(parts[1]);

				// 计算从一天开始的总分钟数 + 延迟分钟数
				let totalMinutes = hours * 60 + minutes + delayMinutes;

				// 处理时间环绕 (e.g., 23:50 + 20min = 00:10, 00:05 - 10min = 23:55)
				const minutesInDay = 24 * 60;

				// 使用取模运算获取最终分钟数，并处理负数结果
				let finalMinutes = totalMinutes % minutesInDay;
				if (finalMinutes < 0) {
					finalMinutes += minutesInDay; // 确保结果在 0 到 1439 之间
				}

				let finalHours = Math.floor(finalMinutes / 60);
				let finalMin = finalMinutes % 60;

				// 格式化为 "HH:mm"
				const formattedHours = String(finalHours).padStart(2, '0');
				const formattedMinutes = String(finalMin).padStart(2, '0');

				return `${formattedHours}:${formattedMinutes}`;
			},

			/**
			 * 根据状态判断并返回状态文本
			 * @param {number|null} delayMinutes 晚点分钟数 (负数为早点)
			 * @param {number|null} status 状态码 (1: 正点, 2: 晚点, 3: 早点)
			 * @returns {string} 状态文本或 '-'
			 */
			formatDelayStatus: function(delayMinutes, status) {
				// 1. 如果 delayMinutes 或 status 为 null/undefined，则为 "-"
				if (delayMinutes === null || status === null || delayMinutes === undefined || status === undefined) {
					return '-';
				}

				// 2. 如果 status 为 1 且 delayMinutes 为 0，则为 "正点"
				if (status === 1 && delayMinutes === 0) {
					return '正点';
				}

				// 3. 如果 status 为 3 且 delayMinutes 为负数，则为 "早点{|delayMinutes|}分" (绿色)
				if (status === 3 && delayMinutes < 0) {
					return `早点${Math.abs(delayMinutes)}分`;
				}

				// 4. 如果 status 为 2 且 delayMinutes 为正数，则为 "晚点{|delayMinutes|}分" (红色)
				if (status === 2 && delayMinutes > 0) {
					return `晚点${delayMinutes}分`;
				}

				// Fallback
				return '-';
			},

			/**
			 * 根据状态返回对应的 CSS 颜色
			 * @param {number|null} delayMinutes 晚点分钟数
			 * @param {number|null} status 状态码
			 * @returns {string} CSS 颜色字符串 (e.g., 'color: #c0392b;')
			 */
			getDelayStatusColor: function(delayMinutes, status) {
				// 早点：绿色
				if (status === 3 && delayMinutes < 0) {
					return 'color: #27ae60; font-weight: bold;';
				}
				// 晚点：红色
				if (status === 2 && delayMinutes > 0) {
					return 'color: #c0392b; font-weight: bold;';
				}
				// 默认（正点或未知）：无特定颜色
				return '';
			},

			/**
			 * 根据状态返回徽章文本和类型
			 * @param {number|null} delayMinutes 晚点分钟数
			 * @param {number|null} status 状态码
			 * @returns {object} {text: string, type: string}
			 */
			getDelayBadgeText: function(delayMinutes, status) {
				// 晚点
				if (status === 2 && delayMinutes > 0) {
					return {
						text: '晚',
						type: 'error' // 红色
					};
				}
				// 早点
				if (status === 3 && delayMinutes < 0) {
					return {
						text: '早',
						type: 'success' // 绿色
					};
				}
				// 正点或未知，不显示徽章
				return {
					text: '',
					type: ''
				};
			},

			fillInData: async function(mode) {
				
				uni.showLoading({
					title: "加载中"
				});
				let loadSuccess = false; // 标记 carData 是否成功加载

				try {
					if (!this.train) return;

					if (mode == "network") {
						const resp = await uniGet(
							`https://data.railgo.zenglingkun.cn/api/train/query?train=${encodeURIComponent(this.train)}`
						);
						const result = resp.data;

						if (result.error || !result.timetable || result.timetable.length === 0) {
							this.carData = {
								numberKind: '',
								numberFull: [],
								type: '',
								timetable: [],
								bureauName: '',
								runner: '',
								carOwner: '',
								car: '',
								rundays: [],
								diagram: []
							};
							this.cardColor = '#114598';
							uni.showToast({
								title: '车次不存在',
								icon: 'error'
							})
							const c = uni.getStorageSync("search");
							uni.setStorage({
								key: 'search',
								data: c - 1
							});
							uni.redirectTo({
								url: '/pages/404/404'
							})
							return; // 结束执行
						}

						// 成功处理
						this.carData = {
							numberKind: result.numberKind || '',
							numberFull: Array.isArray(result.numberFull) ? result.numberFull : [],
							type: result.type || '',
							// 预处理 timetable，确保安全
							timetable: (result.timetable || []).map(item => ({
								station: '',
								stationTelecode: '',
								trainCode: '',
								arrive: '',
								depart: '',
								stopTime: '-',
								distance: '-',
								speed: 0,
								day: '-',
								...item
							})),
							bureauName: result.bureauName || '',
							runner: result.runner || '',
							carOwner: result.carOwner || '',
							car: result.car || '',
							rundays: Array.isArray(result.rundays) ? result.rundays : [],
							diagram: Array.isArray(result.diagram) ? result.diagram : []
						};
						this.cardColor = this.colorMap[this.carData.numberKind] || '#114598';
						loadSuccess = true; // 标记成功

					} else {
						// --- 本地模式逻辑：获取车次详情 ---
						const result = await doQuery("SELECT * FROM trains WHERE number='" + this.keyword +
							"'", KEYS_STRUCT_TRAINS);

						if (result && result.length > 0) {
							// 成功处理
							this.carData = {
								numberKind: '',
								numberFull: [],
								type: '',
								timetable: [],
								bureauName: '',
								runner: '',
								carOwner: '',
								car: '',
								rundays: [],
								diagram: [],
								...toRaw(result[0])
							};
							for (var i = 0; i < this.carData.diagram.length; i++) {
								let dg = toRaw(await doQuery("SELECT code, numberFull FROM trains WHERE number='" + this
									.carData.diagram[i].train_num + "'"))[0];
								if (dg) {
									Object.assign(this.carData.diagram[i], dg);
								}
							}
							this.carData.timetable = (this.carData.timetable || []).map(item => ({
								station: '',
								stationTelecode: '',
								trainCode: '',
								arrive: '',
								depart: '',
								stopTime: '-',
								distance: '-',
								speed: 0,
								day: '-',
								...item
							}));
							this.cardColor = this.colorMap[this.carData.numberKind] || '#114598';
							loadSuccess = true; 

						} else {
							this.carData = {
								numberKind: '',
								numberFull: [],
								type: '',
								timetable: [],
								bureauName: '',
								runner: '',
								carOwner: '',
								car: '',
								rundays: [],
								diagram: []
							};
							this.cardColor = '#114598';
							uni.showToast({
								title: '车次不存在',
								icon: 'error'
							})
							const c = uni.getStorageSync("search");
							uni.setStorage({
								key: 'search',
								data: c - 1
							});
							uni.redirectTo({
								url: '/pages/404/404'
							})
							return; 
						}
					}
					if (this.isOnlyOfflineMode){
						uni.hideLoading()
						return
					}
					if (loadSuccess && this.carData.timetable.length > 0) {
						const timetable = this.carData.timetable;
						const fromStation = timetable[0].station;
						const toStation = timetable[timetable.length - 1].station;

						if (fromStation && toStation && this.date) {
							try {
								const delayResp = await uniPost(
									'https://rail.moefactory.com/api/trainDetails/queryTrainDelayDetails', {
										date: this.date,
										trainNumber: this.train,
										fromStationName: fromStation,
										toStationName: toStation
									}
								);
								if (delayResp.data && Array.isArray(delayResp.data.data)) {
									this.delay = delayResp.data.data;
								} else {
									this.delay = [];
								}
							} catch (delayError) {
								console.warn("获取正晚点信息失败（网络可能断开或接口错误）", delayError);
								this.delay = []; 
							}
						}
					}
					// -------------------------------------------------------------------------

				} catch (error) {
					console.error("数据加载失败", error);
					this.carData = {
						numberKind: '',
						numberFull: [],
						type: '',
						timetable: [],
						bureauName: '',
						runner: '',
						carOwner: '',
						car: '',
						rundays: [],
						diagram: []
					};
					this.cardColor = '#114598';
					this.delay = [];
					const c = uni.getStorageSync("search");
					uni.setStorage({
						key: 'search',
						data: c - 1
					});
				} finally {
					uni.hideLoading();
				}
			},
			tabChange: function(e) {
				this.selectIndex = e.index;
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