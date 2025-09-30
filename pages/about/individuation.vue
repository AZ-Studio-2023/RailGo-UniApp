<template>
	<view class="ux-bg-grey5" style="height:100vh;">
		<view class="ux-bg-primary" style="height:  var(--status-bar-height);">&nbsp;</view>
		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">个性化</text>
		</view>
		<view class="ux-pl ux-pr">
			<uni-section type="line" style="background-color: transparent;" title-font-size="35rpx" title="图标"></uni-section>
			<view class="icon-flex-container">
				<view class="icon-item">
					<image mode="scaleToFill" src="/static/icons/rg-crh.png"></image>
					<text class="icon-text">CRH</text>
					<button class="ux-btn" @click="selectIcon('crh')" :disabled="nowIcon === 'crh'">
						{{ nowIcon === 'crh' ? '已使用' : '使用' }}
					</button>
				</view>

				<view class="icon-item">
					<image mode="scaleToFill" src="/static/icons/rg-red.png"></image>
					<text class="icon-text">红神龙</text>
					<button class="ux-btn" @click="selectIcon('red')" :disabled="nowIcon === 'red'">
						{{ nowIcon === 'red' ? '已使用' : '使用' }}
					</button>
				</view>

				<view class="icon-item">
					<image mode="scaleToFill" src="/static/icons/rg-gold.png"></image>
					<text class="icon-text">金凤凰</text>
					<button class="ux-btn" @click="selectIcon('gold')" :disabled="nowIcon === 'gold'">
						{{ nowIcon === 'gold' ? '已使用' : '使用' }}
					</button>
				</view>

				<view class="icon-item">
					<image mode="scaleToFill" :src="'/static/icons/rg-green.png'"
						:class="{'grayscale': !eggUnlock}"></image>
					<text class="icon-text">立及甬</text>
					<button class="ux-btn" :disabled="!eggUnlock || nowIcon === 'green'" @click="selectIcon('green')">
						{{ nowIcon === 'green' ? '已使用' : (eggUnlock ? '使用' : '未解锁') }}
					</button>
				</view>

				<view class="icon-item">
					<image mode="scaleToFill" :src="'/static/icons/rg-purple.png'"
						:class="{'grayscale': !searchUnlock1}"></image>
					<text class="icon-text">大茄子</text>
					<button class="ux-btn" :disabled="!searchUnlock1 || nowIcon === 'purple'" @click="selectIcon('purple')">
						{{ nowIcon === 'purple' ? '已使用' : (searchUnlock1 ? '使用' : '未解锁') }}
					</button>
				</view>
				<view class="icon-item">
					<image mode="scaleToFill" :src="'/static/icons/rg-passion.png'"
						:class="{'grayscale': !searchUnlock2}"></image>
					<text class="icon-text">百香果</text>
					<button class="ux-btn" :disabled="!searchUnlock2 || nowIcon === 'passion'" @click="selectIcon('passion')">
						{{ nowIcon === 'passion' ? '已使用' : (searchUnlock2 ? '使用' : '未解锁') }}
					</button>
				</view>

				<view class="icon-item">
					<image mode="scaleToFill" src="/static/icons/rg-girl.png" :class="{'grayscale': !girlUnlock}" @click="showGirlHelp"></image>
					<text class="icon-text">RailGo娘</text>
					<button class="ux-btn" @click="selectIcon('girl')" :disabled="!girlUnlock || nowIcon === 'girl'">
						{{ nowIcon === 'girl' ? '已使用' : (girlUnlock ? '使用' : '未解锁') }}
					</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	// #ifdef APP
	import { getSwitchList, switchIcons, restoreIcons } from "@/uni_modules/ima-icons";
	// #endif
import {uniGet} from "@/scripts/req.js";

	export default {
		data() {
			return {
				items: [],
				egg: uni.getStorageSync("Funnyegg"),
				search: uni.getStorageSync("search"),
				nowIcon: uni.getStorageSync("nowIcon"),
				eggUnlock: false,
				searchUnlock1: false,
				searchUnlock2: false,
				girlUnlock: false,
				dfh: uni.getStorageSync("dfh"),
				tys: uni.getStorageSync("tys"),
			};
		},
		mounted() {
			this.updateUnlockStatus();
		},
		methods: {
			back: function() {
				uni.navigateBack()
			},
			updateUnlockStatus() {
				this.eggUnlock = this.egg === true;
				this.searchUnlock1 = this.search >= 50;
				this.searchUnlock2 = this.search >= 100;
				this.girlUnlock = this.dfh === true && this.tys === true;
			},
			selectIcon(iconName) {

				console.log('你选择了图标：' + iconName);
				uni.setStorage({
					"key": "nowIcon",
					"data": iconName
				});
				this.nowIcon = iconName;
				// #ifdef APP
				d = switchIcons(iconName);
				uni.showToast({
					title: "切换成功！",
					icon: 'none'
				});
				// #endif
			},
			showIconHelp() {
				uni.showModal({
					title: '图标解锁说明',
					content: '立及甬需要发现菜单后可解锁。\n大茄子在查询50次后解锁。\n百香果在查询100次后解锁。',
					showCancel: false,
					confirmText: '我知道了'
				});
			},
			// 新增RailGo娘的解锁提示弹窗
			showGirlHelp() {
				if (!this.girlUnlock) {
					uni.showModal({
						title: '提示',
						content: '2025-09-14 ~ 2025.09.30日\n在车站查询查询东方红、太阳升车站，即可解锁',
						showCancel: false,
						confirmText: '我知道了'
					});
				}
			}
		}
	};
</script>

<style>
	/* 样式部分保持不变 */
	.content {
		padding: 20px;
		text-align: center;
	}

	.thanks-text {
		font-size: 32px;
		font-weight: bold;
		color: #333;
		margin-top: 30px;
	}

	.developer-list {
		margin-top: 20px;
	}

	.developer-name {
		display: block;
		font-size: 18px;
		color: #555;
		margin: 5px 0;
	}
	
	.section-title-container {
		display: flex;
		align-items: center;
	}
	
	.uni-section-title {
		font-size: 35rpx;
		font-weight: bold;
		margin-right: 10rpx;
	}

	.icon-flex-container {
		display: flex;
		flex-wrap: wrap;
		gap: 20px;
		justify-content: center;
		padding: 20px 0;
	}

	.icon-item {
		flex: 1 1 200rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		text-align: center;
		min-width: 150rpx;
	}

	.icon-item image {
		width: 150rpx;
		height: 150rpx;
		margin-bottom: 5px;
		transition: filter 0.3s ease;
	}

	.icon-text {
		font-size: 24rpx;
		color: #666;
	}

	.ux-btn {
		width: 150rpx;
		margin-top: 10px;
		font-size: 24rpx;
		line-height: 2;
	}

	.grayscale {
		filter: grayscale(100%);
	}

	.ux-btn[disabled] {
		background-color: #ccc;
		color: #666;
	}
</style>