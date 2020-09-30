<template>
	<view>
		<page-head :title="title"></page-head>
		<view class="uni-common-mt">
			<view class="uni-list">
				<view class="uni-list-cell">
					<view class="uni-pd">
						<view class="uni-label" style="width:180px;">电量</view>
					</view>
					<view class="uni-list-cell-db">
						<input class="uni-input" type="text" :disabled="true" placeholder="未获取" :value="level"></input>
					</view>
				</view>
				<view class="uni-list-cell">
					<view class="uni-pd">
						<view class="uni-label" style="width:180px;">充电状态</view>
					</view>
					<view class="uni-list-cell-db">
						<input class="uni-input" type="text" :disabled="true" placeholder="未获取" :value="status"></input>
					</view>
				</view>
			</view>
		</view>
		<view>
			<view>
				<view class="uni-label" style="width:180px;">铃声</view>
			</view>
			<view class="uni-list-cell-db">
				<picker @change="bindPickerChange" :value="index" :range="mp3Arr" range-key="title">
					<view class="uni-input">{{mp3Arr[index].title}}</view>
				</picker>
			</view>
		</view>
	</view>
</template>
<script>
	let bgAudioMannager = uni.getBackgroundAudioManager();
	bgAudioMannager.title = '致爱丽丝';
	bgAudioMannager.singer = '暂无';
	bgAudioMannager.coverImgUrl = 'https://img-cdn-qiniu.dcloud.net.cn/uniapp/audio/music.jpg';
	export default {
		data() {
			return {
				title: 'getChargStatus',
				status: "",
				level: 0,
				mp3Arr: [{
					title: "hasaki1",
					src: "../../static/mp3/hasaki.mp3"
				}, {
					title: "daomei",
					src: "../../static/mp3/daomei.mp3"
				}, {
					title: "test",
					src: "../../static/mp3/test.mp3"
				}],
				mp3Src: "",
				index: 0,
			}
		},
		onLoad: function() {
			const that = this;
			if(uni.getStorageSync("mp3Src") !== ""){
				uni.setStorageSync("mp3Src", this.mp3Arr[0].src);
				this.mp3Src = uni.getStorageSync("mp3Src");
				bgAudioMannager.src = this.mp3Src;
			}else{
				this.mp3Src = uni.getStorageSync("mp3Src");
				bgAudioMannager.src = this.mp3Src;
			}
			if (uni.getStorageSync("chargeStatus") !== "") {
				uni.setStorageSync("chargeStatus", "discharging")
			}
			this.timer = setInterval(() => {
				that.listen_Battery();
			}, 1000)
		},
		onUnload: function() {
			this.status = "";
			this.level = 0;
		},
		watch: {
			status: (d) => {
				//	1.0.0版本  未充电前打开提示音可以充电  打开后不行 需要监听充电状态变化 提供切换充电提示音功能 充电提示音要添加可以切换并支持试听功能
				let localStatus = uni.getStorageSync("chargeStatus");
				if (localStatus !== "charging") {
					if (d === "charging") {
						uni.setStorageSync("chargeStatus", "charging")
						bgAudioMannager.play();
					}
				} else {
					if (d !== "charging") {
						uni.setStorageSync("chargeStatus", d)
					}
				}
			}
		},
		methods: {
			bindPickerChange(e) {
				this.index = e.target.value;
				this.mp3Src = this.mp3Arr[this.index].src;
				console.log(JSON.stringify(bgAudioMannager))
				bgAudioMannager.src = this.mp3Src;
				uni.setStorageSync("mp3Src", this.mp3Src);
				bgAudioMannager.play();
			},
			listen_Battery: function() {
				const _this = this;
				try {
					var main = plus.android.runtimeMainActivity(); //获取activity  
					var Intent = plus.android.importClass('android.content.Intent');
					var receiver;
					receiver = plus.android.implements('io.dcloud.feature.internal.reflect.BroadcastReceiver', {
						onReceive: function(context, intent) { //实现onReceiver回调函数  
							try {
								var BatteryManager = plus.android.importClass('android.os.BatteryManager');
								var action = intent.getAction();
								if (action == Intent.ACTION_BATTERY_CHANGED) {
									var status = intent.getIntExtra("status", 0);
									var level = intent.getIntExtra("level", 0);
									var plugged = intent.getIntExtra("plugged", 0);

									var statusString = "";
									switch (status) {
										case BatteryManager.BATTERY_STATUS_UNKNOWN:
											statusString = "unknown";
											break;
										case BatteryManager.BATTERY_STATUS_CHARGING:
											statusString = "charging";
											break;
										case BatteryManager.BATTERY_STATUS_DISCHARGING:
											statusString = "discharging";
											break;
										case BatteryManager.BATTERY_STATUS_NOT_CHARGING:
											statusString = "not charging";
											break;
										case BatteryManager.BATTERY_STATUS_FULL:
											statusString = "full";
											break;
									}
									var acString = "";
									switch (plugged) {
										case BatteryManager.BATTERY_PLUGGED_AC:
											acString = "plugged ac";
											break;
										case BatteryManager.BATTERY_PLUGGED_USB:
											acString = "plugged usb";
											break;
									}
									_this.status = statusString;
									// console.log(_this.status)
									_this.level = level;
									main.unregisterReceiver(receiver);
								}
							} catch (e1) {
								console.log(e1);
							}
						}
					});
					var IntentFilter = plus.android.importClass('android.content.IntentFilter');
					var filter = new IntentFilter();
					filter.addAction(Intent.ACTION_BATTERY_CHANGED);
					main.registerReceiver(receiver, filter); //注册监听  
				} catch (e) {
					console.log(e);
				}
			}
		}
	}
</script>

<style>
	.uni-pd {
		padding-left: 30rpx;
	}
</style>
