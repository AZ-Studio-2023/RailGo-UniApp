<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		<view class="ux-bg-primary" style="height: height: var(--status-bar-height);">&nbsp;</view>

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
				<uni-section title="时刻表" type="line" style="background-color: transparent;"
					title-font-size="25rpx"></uni-section>
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
					title-font-size="25rpx"></uni-section>
				<uni-table border stripe>
					<uni-tr>
						<uni-th align="center">车站</uni-th>
						<uni-th align="center">车次</uni-th>
						<uni-th align="center">到达时间<br>发车时间</uni-th>
						<uni-th align="center">停站时间</uni-th>
						<uni-th align="center">日期</uni-th>
						<uni-th align="center">状态</uni-th>
					</uni-tr>
					<uni-tr v-for="(item,index) in delay" :key="index">
						<uni-td>{{item.stationName || ''}}</uni-td>
						<uni-td>{{item.trainNumber || ''}}</uni-td>
						<uni-td>{{item.arrivalTime || ''}}<br>{{item.departureTime || ''}}</uni-td>
						<uni-td>{{item.arrivalDate || ''}}</uni-td>
						<uni-td>{{item.status || ''}}</uni-td>
					</uni-tr>
				</uni-table>
			</view>

			<view v-if="selectIndex==1">
				<uni-section title="担当" type="line" style="background-color: transparent;"
					title-font-size="25rpx"></uni-section>
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
									style="max-width:350rpx; height: 200rpx; border-radius: 30%; overflow: hidden;"></image>
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
					title-font-size="25rpx"></uni-section>
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
					title-font-size="25rpx"></uni-section>
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
	import uniGet from "@/scripts/req"; // 导入网络请求
	
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
					diagram: []
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
				"selectIndex": 0
			}
		},
		onLoad(options) {
			this.train = options.keyword ? options.keyword.split("/")[0] : '';
			this.keyword = options.keyword;
			this.title = this.train;
			this.date = options.date || '';

			const mode = uni.getStorageSync("mode");
			
			const c = uni.getStorageSync("search");
			uni.setStorage({
				key: 'search',
				data: c+1
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
			fillInData: async function(mode) {
				uni.showLoading({
				    title: "加载中"
				});
				if (mode == "network") {
					// --- 网络模式逻辑 ---
					try {
						if (!this.train) return;
						const resp = await uniGet(`https://data.railgo.zenglingkun.cn/api/train/query?train=${encodeURIComponent(this.train)}`);
						const result = resp.data;
						if (result.error) {
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
							uni.hideLoading()
							uni.showToast({
								title: '车次不存在',
								icon: 'error'
							})
							const c = uni.getStorageSync("search");
							uni.setStorage({
								key: 'search',
								data: c-1
							});
							uni.navigateBack()
							return;
						}
						// 处理字段，确保安全
						this.carData = {
							numberKind: result.numberKind || '',
							numberFull: Array.isArray(result.numberFull) ? result.numberFull : [],
							type: result.type || '',
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
						uni.hideLoading()
					} catch (error) {
						console.error("数据加载失败", error);
						uni.hideLoading()
						const c = uni.getStorageSync("search");
						uni.setStorage({
							key: 'search',
							data: c-1
						});
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
					}
				} else {
					// --- 本地模式逻辑 ---
					try {
						if (!this.keyword) {
							return;
						}
						const result = await doQuery("SELECT * FROM trains WHERE number='" + this.keyword +
							"'", KEYS_STRUCT_TRAINS);
						if (result && result.length > 0) {
							// 修正：确保 carData 所有字段都有默认值，防止页面报错
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
									// 修正：合并数据而不是只覆盖
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
						} else {
							// 如果没有查询结果，也需要重置数据并给出提示
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
								data: c-1
							});
							uni.navigateBack();
							return;
						}
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
					} finally {
						uni.hideLoading();
					}
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