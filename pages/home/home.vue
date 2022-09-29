<template>
	<view>
		<view class="watermark" v-if="!cameraFlag">
			<view class="img-item" v-for="(item,index) in imgList" :key="index">
				<image :src="item" mode="aspectFit" style="width: 100%;height: 100%;" @click="preview(index)"></image>
				<uni-icons class="del-btn" type="clear" size="20" color="#FF3434" @click="del(index)"></uni-icons>
			</view>
			<view class="add-image-btn" @click="handleCamera">
				<uni-icons type="camera-filled" size="40" color="#C6C6C6"></uni-icons>
			</view>
		</view>
		<!-- 相机模块 -->
		<WatermarkCamera v-else @complete="complete" @back="handleCamera" :markText="markText" :bgMark="bgMark" :bgMarkText="bgMarkText"/>
	</view>
</template>

<script>
	import WatermarkCamera from "@/components/WatermarkCamera/WatermarkCamera.vue"
	export default {
		data() {
			return {
				imgList: [],
				cameraFlag: false,
				address: "",
				key: "", //地图API
				markText:"",//水印信息
				bgMark: true,
				bgMarkText: ""
			};
		},
		components: {
			WatermarkCamera
		},
		mounted() {
			console.log("页面加载中...")
			const this_ = this
			//获取定位
			uni.getLocation({
				type: 'gcj02',
				wgs84: false,
				sHighAccuracy: false,
				highAccuracyExpireTime: 3000,
				success: function(res) {
					console.log('成功获取位置信息', res.latitude)
					uni.request({
						url: `https://apis.map.qq.com/ws/geocoder/v1/?location=${res.latitude},${res.longitude}&key=${this_.key}`, //调用地图接口，随便选
						success: (res) => {
							// this_.address = res.data.result.formatted_addresses.rough
							this_.markText = res.data.result.formatted_addresses.rough
							// this_.markInfos.fillColor = "red"
						}
					})
				},
				fail: function(error) {
					console.log('获取当前位置失败', error)
				},
				complete: function(com) {
					console.log('接口调用结束的回调函数（调用成功、失败都会执行）', com)
				}
			});
			
			/*************** 获取日期与时间 ***************/
			// let time = _this.$u.timeFormat(new Date(),
			// 	'hh:MM')
			this.bgMarkText = this.$u.timeFormat(new Date(),
				'yyyy.mm.dd')
			// context.setFontSize(70);
			// context.fillText(time, 20, image.height - 300)
			// context.setFontSize(25);
			// context.fillText(date, 210, image.height - 300)
			// let week = "星期" + "日一二三四五六".charAt(new Date().getDay())
			// context.fillText(week, 210, image.height - 330)
			
		},
		methods: {
			//照片处理函数
			complete(data) {
				this.imgList.push(data)
				this.cameraFlag = false
			},
			del(index) {
				this.imgList.splice(index, 1)
			},
			//返回，取消拍照
			handleCamera(e) {
				this.cameraFlag = !this.cameraFlag
			},
			//预览
			preview() {
				uni.previewImage({
					indicator: "number",
					loop: true,
					urls: this.imgList
				})
			}
		},

	}
</script>

<style lang="scss" scoped>
	.watermark {
		display: flex;
		flex-wrap: wrap;
		width: 90%;
		margin: 24rpx auto;

		.img-item,
		.add-image-btn {
			width: 200rpx;
			height: 200rpx;
			background-color: #F0F2F5;
			margin: 0 24rpx 24px 0;
			border-radius: 10rpx;
		}

		.img-item {
			position: relative;

			.del-btn {
				position: absolute;
				right: -10rpx;
				top: -10rpx;
			}
		}

		.add-image-btn {
			display: flex;
			justify-content: center;
			align-items: center;
		}
	}
</style>
