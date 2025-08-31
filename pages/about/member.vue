<template>
	<view class="ux-bg-grey5" style="height:100vh;">
		<!-- headers begin -->
		<view class="ux-bg-primary" style="height: 50rpx;">&nbsp;</view>

		<view class="ux-padding">
			<view hover-class="ux-bg-grey8" @click="back">
				<text class="icon" style="font-size: 45rpx;">&#xe5c4;</text>
			</view>
			<br>
			<text class="ux-h2">鸣谢</text>
		</view>
		<!-- headers end -->

		<!-- Main content -->
		<view class="ux-pl ux-pr">
			<uni-section title="开发" type="line" style="background-color: transparent;"
				title-font-size="35rpx"></uni-section>
			<view class="ux-flex ux-space-around ux-justify-content-center">
				<view class="ux-text-center ux-pl ux-pr">
					<image mode="scaleToFill" src="https://q1.qlogo.cn/g?b=qq&s=100&nk=1316280182"
						style="width:100rpx;height:100rpx;border-radius:50%;"></image>
					<br>
					<text class="ux-text-small">辰墨</text>
				</view>
				<view class="ux-text-center ux-pl ux-pr">
					<image mode="scaleToFill" src="https://q1.qlogo.cn/g?b=qq&s=100&nk=572747232"
						style="width:100rpx;height:100rpx;border-radius:50%;"></image>
					<br>
					<text class="ux-text-small">TKP30</text>
				</view>
				<view class="ux-text-center ux-pl ux-pr">
					<image mode="scaleToFill" src="https://q1.qlogo.cn/g?b=qq&s=100&nk=1219439938"
						style="width:100rpx;height:100rpx;border-radius:50%;"></image>
					<br>
					<text class="ux-text-small">姜胤</text>
				</view>
				<view class="ux-text-center ux-pl ux-pr">
					<image mode="scaleToFill" src="https://q1.qlogo.cn/g?b=qq&s=100&nk=3776473661"
						style="width:100rpx;height:100rpx;border-radius:50%;"></image>
					<br>
					<text class="ux-text-small">mstouk57g</text>
				</view>
			</view>
		</view>
		<view class="ux-pl ux-pr">
			<uni-section title="特别鸣谢" type="line" style="background-color: transparent;"
				title-font-size="35rpx"></uni-section>
			<view class="ux-flex ux-space-around ux-justify-content-center">
				<view class="ux-text-center ux-pl ux-pr">
					<image mode="scaleToFill" src="https://q1.qlogo.cn/g?b=qq&s=100&nk=3068342155"
						style="width:100rpx;height:100rpx;border-radius:50%;"></image>
					<br>
					<text class="ux-text-small">XieXiLin</text>
				</view>
				<view class="ux-text-center ux-pl ux-pr">
					<image mode="scaleToFill" src="https://q1.qlogo.cn/g?b=qq&s=100&nk=2396807944"
						style="width:100rpx;height:100rpx;border-radius:50%;"></image>
					<br>
					<text class="ux-text-small">bcdt</text>
				</view>
			</view>
		</view>
		<view class="ux-pl ux-pr">
			<uni-section title="其他" type="line" style="background-color: transparent;"
				title-font-size="35rpx"></uni-section>
			<view class="ux-flex ux-space-around ux-justify-content-center">
				<view class="ux-text-center ux-pl ux-pr">
					慈云数据 <br> 提供云计算支持
				</view>
			</view>
		</view>
		<view class="ux-pl ux-pr">
			<uni-section title="动车组图片" type="line" style="background-color: transparent;"
				title-font-size="35rpx"></uni-section>
			<image mode="scaleToFill" v-for="(nk, index) in picgive" :src="`https://q1.qlogo.cn/g?b=qq&s=100&nk=${nk}`"
				style="width:90rpx;height:90rpx;border-radius:50%;" class="ux-ml-small ux-r-small"></image>
		</view>
		<view class="ux-pl ux-pr">
			<uni-section title="反馈" type="line" style="background-color: transparent;"
				title-font-size="35rpx"></uni-section>
			<image mode="scaleToFill" v-for="(nk, index) in items" :src="`https://q1.qlogo.cn/g?b=qq&s=100&nk=${nk}`"
				style="width:80rpx;height:80rpx;border-radius:50%;" class="ux-ml-small ux-r-small"></image>
		</view>
	</view>
</template>
<script>
	import uniGet from "@/scripts/req.js";

	export default {
		// Railgo Code
		data() {
			return {
				items: [],
				picgive: ["1972867189", "2151235831", "1069943593", "525637945", "3596323062", "2377807972", "982588465", "1779378782", "516127399", "2533221373", "2387845184", "365736548", "1983163227", "3111737636"]
			};
		},
		mounted() {
			this.fetchData();
		},
		methods: {
			back: function() {
				uni.navigateBack()
			},
			async fetchData() {
				try {
					const Response = await uniGet("https://feedback.railgo.dev/api/get_users");
					this.items = Response.data.users;
				} catch (error) {
					console.error('Error fetching data:', error);
					this.items = [];
				}
			}
		}
	};
</script>

<style>
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
</style>