<template>
	<view class="content">
		<view class="title">{{ title }}</view>
		<view class="section">
			<button @click="initSDK">初始化SDK</button>
			<button @click="checkAndRequestPermission">检查并请求权限</button>
		</view>

		<view class="section">
			<button @click="startDiscover">开始搜索设备</button>
			<button @click="stopDiscover">停止搜索</button>
			<view class="device-list">
				<view v-for="(device, index) in devices" :key="index" class="device-item" @click="connectDevice(device)">
					<text>设备名称: {{device.name || '未知设备'}}</text>
					<text>地址: {{device.address}}</text>
				</view>
			</view>
		</view>

		<view class="section" v-if="isConnected">
			<button @click="getDeviceStatus">获取设备状态</button>
			<button @click="getDeviceInfo">获取设备信息</button>
			<button @click="getMixedStatus">获取混合状态</button>
			<button @click="disconnectDevice">断开连接</button>
		</view>

		<view class="section" v-if="isConnected">
			<button @click="selectAndPrint">选择文件并打印</button>
			<view v-if="currentJobId">
				<text>当前任务ID: {{currentJobId}}</text>
				<button @click="getJobInfo">获取任务信息</button>
				<button @click="cancelJob">取消任务</button>
				<button @click="resumeJob">恢复任务</button>
			</view>
			<button @click="cleanDevice">清理设备</button>
		</view>

		<view class="log-section">
			<text>日志输出:</text>
			<scroll-view class="log-content" scroll-y>
				<text v-for="(log, index) in logs" :key="index">{{log}}</text>
			</scroll-view>
		</view>
		
		<view class="footer">
			<view >Copyright © BSF 2025</view>
			<view class="link" @click="openUrl">https://byteee.fund</view>
		</view>
	</view>
</template>

<script>
	import * as AvocadoSDK from "../../uni_modules/bsf-avocado";
	
	export default {
		data() {
			return {
					title: "bsf-avocado示例工程",
					devices: [],
					isConnected: false,
					currentJobId: null,
					logs: [],
					currentDevice: null
			}
		},
		methods: {
			log(message) {
				this.logs.push(`${new Date().toLocaleTimeString()}: ${message}\n`);
			},

			initSDK() {
				const result = AvocadoSDK.initSDK("9ZWI3PGTRIJE");
				this.log(`SDK初始化: ${result ? '成功' : '失败'}`);
			},

			checkAndRequestPermission() {
				if (!AvocadoSDK.checkPermission()) {
					AvocadoSDK.requestPermission({
						onPermit: () => {
							this.log("权限获取成功");
						},
						onRefuse: () => {
							this.log("权限被拒绝");
						}
					});
				} else {
					this.log("已具备所需权限");
				}
			},

			startDiscover() {
				this.devices = [];
				AvocadoSDK.discover({
					onDiscoveryStarted: () => {
						this.log("开始搜索设备");
					},
					onDeviceFound: (device) => {
						this.devices.push(device);
						this.log(`发现设备: ${device.name || '未知设备'}`);
					},
					onDiscoveryStopped: () => {
						this.log("停止搜索设备");
					}
				});
			},

			stopDiscover() {
				AvocadoSDK.stopDiscover();
			},

			connectDevice(device) {
				this.currentDevice = device;
				AvocadoSDK.connectDevice({
					address: device.address,
					onConnect: (success) => {
						this.isConnected = success;
						this.log(`设备连接${success ? '成功' : '失败'}`);
					}
				});
			},

			disconnectDevice() {
				AvocadoSDK.disconnetDevice();
				this.isConnected = false;
				this.currentDevice = null;
				this.log("设备已断开连接");
			},

			getDeviceStatus() {
				AvocadoSDK.getDeviceStatus({
					onResponse: (response) => {
						this.log(`设备状态: ${response}`);
					},
					onErrorResponse: (errorCode) => {
						this.log(`获取设备状态错误: ${errorCode}`);
					}
				});
			},

			getDeviceInfo() {
				AvocadoSDK.getDeviceInfo({
					onResponse: (response) => {
						this.log(`设备信息: ${response}`);
					},
					onErrorResponse: (errorCode) => {
						this.log(`获取设备信息错误: ${errorCode}`);
					}
				});
			},

			getMixedStatus() {
				AvocadoSDK.getMixedStatus({
					onResponse: (response) => {
						this.log(`混合状态: ${response}`);
					},
					onErrorResponse: (errorCode) => {
						this.log(`获取混合状态错误: ${errorCode}`);
					}
				});
			},

			async selectAndPrint() {
				try {
					const filePath = await uni.chooseFile({
						count: 1,
						extension: ['.jpg', '.png', '.pdf']
					});
					
					AvocadoSDK.printJob({
						tempPath: filePath[0].path,
						copies: 1,
						isFirmUpdata: false,
						onCreateJobSuccess: (jobId) => {
							this.currentJobId = jobId;
							this.log(`创建打印任务成功，任务ID: ${jobId}`);
						},
						onCreateJobFail: (errorCode, errorReason) => {
							this.log(`创建打印任务失败: ${errorReason}`);
						},
						onProgressChange: (max, current, jobId) => {
							this.log(`打印进度: ${current}/${max}`);
						},
						onTransferCompleted: (jobId) => {
							this.log(`任务${jobId}传输完成`);
						},
						onResponse: (response) => {
							this.log(`打印响应: ${response}`);
						},
						onErrorResponse: (errorCode) => {
							this.log(`打印错误: ${errorCode}`);
						}
					});
				} catch (e) {
					this.log('选择文件失败');
				}
			},

			getJobInfo() {
				if (!this.currentJobId) return;
				AvocadoSDK.getJobInfo(this.currentJobId, {
					onResponse: (response) => {
						this.log(`任务信息: ${response}`);
					},
					onErrorResponse: (errorCode) => {
						this.log(`获取任务信息错误: ${errorCode}`);
					}
				});
			},

			cancelJob() {
				if (!this.currentJobId) return;
				AvocadoSDK.cancleJob(this.currentJobId, {
					onResponse: (response) => {
						this.log(`取消任务成功`);
						this.currentJobId = null;
					},
					onErrorResponse: (errorCode) => {
						this.log(`取消任务错误: ${errorCode}`);
					}
				});
			},

			resumeJob() {
				AvocadoSDK.resume({
					onResponse: (response) => {
						this.log(`恢复任务成功`);
					},
					onErrorResponse: (errorCode) => {
						this.log(`恢复任务错误: ${errorCode}`);
					}
				});
			},

			cleanDevice() {
				AvocadoSDK.clean({
					onResponse: (response) => {
						this.log(`清理设备成功`);
					},
					onErrorResponse: (errorCode) => {
						this.log(`清理设备错误: ${errorCode}`);
					}
				});
			},
			openUrl() {
				// #ifdef APP-PLUS
				plus.runtime.openURL('https://byteee.fund', (err) => {
					if (err) {
						uni.showToast({
							title: '打开链接失败',
							icon: 'none'
						});
					}
				});
				// #endif
				
				// #ifdef H5
				window.open('https://byteee.fund', '_blank');
				// #endif
			},
		}
	}
</script>

<style>
	.content {
		padding: 15px;
	}
	
	.title {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
		padding: 20rpx;
		text-align: center;
	}

	.section {
		margin-bottom: 20px;
		padding: 10px;
		border: 1px solid #eee;
		border-radius: 5px;
	}

	.device-list {
		max-height: 200px;
		overflow-y: auto;
	}

	.device-item {
		padding: 10px;
		border-bottom: 1px solid #eee;
		display: flex;
		flex-direction: column;
	}

	.log-section {
		margin-top: 20px;
	}

	.log-content {
		height: 200px;
		border: 1px solid #eee;
		padding: 10px;
		font-size: 12px;
	}

	button {
		margin: 5px 0;
	}
	
	.footer {
			font-size: 24rpx;
			color: #999;
			padding: 20rpx 20rpx 5rpx;
			text-align: center;
		}
	
		.link {
			font-size: 24rpx;
			color: #007aff;
			padding: 5rpx 20rpx;
			text-align: center;
			text-decoration: underline;
		}
	
		.link:active {
			opacity: 0.7;
		}
</style>
