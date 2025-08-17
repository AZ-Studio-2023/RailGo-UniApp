<template>
	<view class="ux-bg-grey5 page">
		<!-- headers begin -->
		<view class="ux-bg-primary status-bar"></view>
		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<view
				class="ux-bg-white ux-border-radius-small ux-padding-small ux-flex ux-align-items-center ux-mt-small ux-mb-small">
				<text class="icon">&#xe8b6;</text>
				&nbsp;
				<input type="text" class="ux-form-input" style="width:100%;margin:0 0;" placeholder="搜索车站名"
					@input="inputData" v-model="keyword" />
			</view>
			<view class="ux-border-radius">
				<uni-list>
					<uni-list-item v-for="(item,index) in stationsList" :key="index" class="ux-bg-white" clickable
						@click="clickToSelect(item, $event)">
						<template v-slot:header>
							<view class="ux-flex ux-align-items-center" style="flex-direction: row;">
								<view v-for="(tag,i) in item.type" :key="i">
									<text class="ux-badge ux-color-white ux-mr-small"
										:style="'background-color:'+this.badgeFlag[tag]">{{tag}}</text>
								</view>
							</view>
						</template>
						<template v-slot:body>
							<text class="ux-flex ux-align-items-center ux-flex1">{{item.name}}站</text>
						</template>
						<template v-slot:footer>
							<view class="ux-flex ux-align-items-center ux-justify-content-end"
								style="flex-direction: row;">
								<text class="ux-text-small ux-opacity-5">{{item.pinyinTriple}}/-{{item.telecode}}</text>
							</view>
						</template>
					</uni-list-item>
				</uni-list>

			</view>
		</view>
		<!-- headers end -->
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
	export default {
		data() {
			return {
				stationsUsed: [],
				stationsList: [],
				keyword: "",
				badgeFlag: {
					"客": "#114598",
					"货": "#eeba67",
					"高": "#c0392b",
					"行": "#459811",
					"运": "#5499c7"
				},
				resultPlace: ""
			};
		},
		onLoad(options) {
			this.resultPlace = options.resultPlace;
			this.fillInData(); // 调用数据填充方法
		},
		methods: {
			back: function() {
				uni.navigateBack()
			},
			inputData: function(e) {
				this.keyword = e.detail.value;
				this.fillInData();
			},
			fillInData: async function() {
				if (this.keyword == "") {
					return;
				}
				this.stationsList = toRaw(await doQuery(
					"SELECT name, telecode, pinyin, pinyinTriple, type FROM stations WHERE name LIKE '%" + this
					.keyword +
					"%'", ["name", "telecode", "pinyin", "pinyinTriple", "type"])).sort((a, b) => {
					// 按拼音排序
					if (a.pinyin < b.pinyin) {
						return -1;
					}
					if (a.pinyin > b.pinyin) {
						return 1;
					}
					return 0;
				});
			},
			clickToSelect: function(item, e) {
				uni.setStorageSync(this.resultPlace, item);
				uni.navigateBack();
			}
		}
	}
</script>

<style>
</style>