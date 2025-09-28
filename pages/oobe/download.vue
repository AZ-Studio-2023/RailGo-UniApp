<template>
	<view class="ux-bg-grey6" style="min-height:100vh;">
		<view class="ux-bg-primary"  style="height:  var(--status-bar-height);">&nbsp;</view>
		<view class="ux-padding-large">
			<view class="ux-mt-large">
				<text class="ux-h1">马上开始...</text><br><br>
				<text class="ux-h6">正在下载离线数据库。</text><br><br>
				<text>{{err}}</text>
				<progress :percent="downloadProgress" show-info stroke-width="3" activeColor="#114598" />
			</view>
		</view>
	</view>
</template>
<script>
import {uniGet} from "@/scripts/req.js";
	import {
		loadDB
	} from "@/scripts/sqlite.js";
	
	const DB_PATH = '_doc/railgo.sqlite';

	const saveFileAsync = (tempFilePath) => {
		return new Promise((resolve, reject) => {
			uni.saveFile({
				tempFilePath: tempFilePath,
				success: res => resolve(res),
				fail: err => reject(err)
			});
		});
	};

	const decompressAsync = (filePath, targetDir) => {
		return new Promise((resolve, reject) => {
			// 注意：plus.zip.decompress 只有在 APP-PLUS 环境下才存在
			plus.zip.decompress(filePath, targetDir, resolve, reject);
		});
	};
	
	/**
	 * 检查数据库文件是否存在
	 * @returns {Promise<boolean>}
	 */
	const checkDBFileExists = () => {
		return new Promise(resolve => {
			// plus.io.resolveLocalFileSystemURL 只有在 APP-PLUS 环境下才存在
			plus.io.resolveLocalFileSystemURL(DB_PATH, 
				// 成功回调：文件或目录存在
				() => resolve(true),
				// 失败回调：文件或目录不存在
				() => resolve(false)
			);
		});
	};

	export default {
		data() {
			return {
				downloadProgress: 0,
				err: ""
			}
		},
		async onReady() {
		    // #ifdef APP-PLUS
		    try {
				// 1. 检查数据库文件是否已存在
				const isDBExist = await checkDBFileExists();
				let shouldDownload = true;
				
				if (isDBExist) {
					this.err = "数据库文件已存在，跳过下载和解压。";
					this.downloadProgress = 100;
					shouldDownload = false;
				}
				
				// 2. 如果文件不存在，则执行下载和解压流程
				if (shouldDownload) {
					this.err = "正在获取下载地址...";
					const dbUrlResponse = await uniGet("https://api.state.railgo.zenglingkun.cn/api/v1/url/db");
		        	const downloadUrl = dbUrlResponse.data.url;
					
					this.err = "正在下载文件...";
		        	const downloadResult = await new Promise((resolve, reject) => {
		            	const downloadTask = uni.downloadFile({
		                	url: downloadUrl,
		                	success: res => {
		                    	if (res.statusCode === 200) {
		                        	resolve(res);
		                    	} else {
		                        	reject(new Error(`下载失败，状态码: ${res.statusCode}`));
		                    	}
		                	},
		                	fail: err => reject(err)
		            	});
		            	downloadTask.onProgressUpdate((res) => {
		                	this.downloadProgress = res.progress;
		            	});
		        	});
		
		        	// 确保临时文件存在
		        	if (!downloadResult.tempFilePath) {
		            	throw new Error("未获取到临时文件路径");
		        	}
		
		        	// 保存文件
		        	this.err = "正在保存文件...";
		        	const savedFile = await saveFileAsync(downloadResult.tempFilePath);
		        
		        	// 解压文件
		        	this.err = "正在解压文件...";
		        	await decompressAsync(savedFile.savedFilePath, "_doc/");
				}
		        
		        // 3. 文件处理完成后，执行获取信息和加载数据库的后续步骤
		
		        // 获取数据库信息（无论是否下载，都应获取最新版本信息）
		        this.err = "正在获取数据库信息...";
		        const finalInfoResponse = await uniGet("https://api.state.railgo.zenglingkun.cn/api/v1/info");
		        
		        if (finalInfoResponse.status === 200) {
		            uni.setStorageSync("offlineDataVersion", finalInfoResponse.data.latest_db);
		            uni.setStorageSync("offlineDataVersionText", finalInfoResponse.data.db);
		        } else {
		            throw new Error("获取数据库信息失败: " + JSON.stringify(finalInfoResponse.data));
		        }
		
		        // 加载数据库
		        this.err = "正在加载数据库...";
		        await loadDB();
		
		        // 设置完成标志并跳转
		        uni.setStorageSync("oobe", true);
		        uni.setStorageSync("mode", "local");
		        uni.reLaunch({ 
		            url: '/pages/index/index' 
		        });
		
		    } catch (e) {
		        console.error("下载或初始化过程中发生错误:", e);
		        this.err = "发生错误: " + (e.message || JSON.stringify(e));
		        uni.showModal({
		            title: "错误",
		            content: "下载或加载数据库失败: " + e.message,
		            showCancel: false
		        });
		    }
		    // #endif
		
		    // #ifdef H5
		    uni.setStorageSync("oobe", true);
		    uni.setStorageSync("mode", "online");
		    uni.reLaunch({ 
		        url: '/pages/index/index' 
		    });
		    // #endif
		},
		methods: {
		}
	}
</script>

<style>
	.primary-button {
		width: 100rpx;
		height: 100rpx;
		border-radius: 50%;
		text-align: center;
		line-height: 100rpx;
		background-color: #114598;
		color: #fff;
		margin-top: 80rpx;
	}

	.icon {
		font-size: 40rpx;
	}
</style>
