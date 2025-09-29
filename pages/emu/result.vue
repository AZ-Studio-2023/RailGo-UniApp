<template>
	<view class="ux-bg-grey5" style="min-height:100vh;">
		<view class="ux-bg-primary" style="height:  var(--status-bar-height);">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">{{keyword}}</text>
		</view>
		<view class="ux-pl ux-pr ux-pb">
			<view class="ux-text-center ux-padding-small ux-mb ux-h6 ux-text-center"
				style="background-color:#e9eef5;border:1px solid #114598;border-radius:10rpx;color:#114598;">
				<text class="ux-bold">信息仅供参考 请以铁路运营企业实际运用为准</text>
			</view>
			<uni-section style="background-color:#eeeeee;margin-left:-1.2vh;" type="line" title="点击车次或车组可查询对应车辆和车次的担当"></uni-section>
			
			<uni-table border stripe :loading="loading">
				<uni-tr>
						<uni-th align="center">担当日</uni-th>
						<uni-th align="center">车次</uni-th>
						<uni-th align="left">车组</uni-th>
				</uni-tr>
					<uni-tr v-for="(item,index) in data" :key="index">
						<uni-td>{{item.date}}</uni-td>
						<uni-td>
							<navigator :url="getTrainNavigatorURL(item)">
								{{item.train_no}}
							</navigator>
						</uni-td>
						<uni-td>
							<navigator :url="getCarNavigatorURL(item, 0)">
								{{item.emu_no}}
							</navigator>
</uni-td>
					</uni-tr>
			</uni-table>

			<view v-if="data.length === 0 && !loading" class="ux-text-center ux-padding">
				<text class="ux-opacity-6">—— 暂无数据 ——</text>
			</view>
			
			<br>
			<view class="ux-flex ux-row ux-justify-content-center">
				<text class="ux-text-small ux-opacity-4">—— 数据来源: RAIL.RE（临时） ——</text>
			</view>
		</view>
	</view>
</template>

<script>
import {uniGet} from "@/scripts/req.js";
	export default {
		data() {
			return {
				"keyword": "",
				"future": true,
				"data": [],
				"loading": false // 新增 loading 状态
			}
		},
		onLoad(options) {
			this.keyword = options.keyword;
			this.future = options.future;
			this.fillInData();
			const c = uni.getStorageSync("search");
			uni.setStorage({
				key: 'search',
				data: (c || 0) + 1 // 修正了 c+1 的潜在错误，确保 c 是数字
			});
			// 移除 onLoad 中的 uni.showLoading，统一到 fillInData 中管理
		},
		onShow() {
			plus.navigator.setStatusBarBackground('#eeeeee');
		},
		methods: {
			back: function() {
				uni.navigateBack()
			},
			fillInData: async function() {
			    this.loading = true; // 开始加载
			    try {
			        uni.showLoading({ title: '加载中...' }); 
			        
			        const keyword = this.keyword || "";
			        const t = keyword.includes("CR") ? "emu" : "train";
			        
			        const apiUrl = `https://emu.data.railgo.zenglingkun.cn/${t}/${encodeURIComponent(keyword)}`;
			        
			        const response = await uniGet(apiUrl);
			        this.data = response.data;
			        console.log("数据加载成功", this.data);
					
					if (this.data.length === 0) {
						uni.redirectTo({
							url: '/pages/404/404'
						});
					}
			        
			    } catch (error) {
			        console.error("数据加载失败", error);
			        uni.showToast({
			            title: "加载失败",
			            icon: "none",
			            duration: 2000 // 延长显示时间
			        });
			        
			        if (this.data.length === 0) {
			            uni.redirectTo({
			                url: '/pages/404/404'
			            });
			        }

			    } finally {
			        uni.hideLoading(); 
			        this.loading = false; // 结束加载
			    }
			},
			getTrainNavigatorURL: function(item){
				return '/pages/emu/result?keyword='+item.train_no+'&future='+this.future;
			},
			getCarNavigatorURL: function(item, index){
				return '/pages/emu/result?keyword='+item.emu_no+'&future='+this.future;
			}
		}
	}
</script>