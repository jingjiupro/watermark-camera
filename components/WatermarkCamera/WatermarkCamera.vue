<template>
	<view class="watermark-camera">
		<view class="camera-box" v-if="tempSrc==''&&src==''">
			<camera device-position="back" flash="off" @error="error" style="width: 100%; height: 100vh;"></camera>
			<uni-icons class="back-btn" type="bottom" size="30" color="#FFFFFF" @click="back"></uni-icons>
			<view class="photo-outter-btn" @click="takePhoto">
				<view class="photo-inner-btn">
				</view>
			</view>
		</view>
		<view class="img-box" v-if="src!==''">
			<button class="complete-photo" type="primary" size="mini" @click="completePhoto">完成</button>
			<button class="re-photo" type="default" size="mini" @click="rePhoto">重拍</button>
			<image mode="widthFix" :src="src" style="width: 100%;"></image>
		</view>
		<canvas canvas-id="myCanvas" id="myCanvas"
			:style="{width: canvasSiz.width+'px',height: canvasSiz.height+'px',position: 'absolute',left:'-500000px',top: '-500000px'}" />
	</view>
</template>

<script>
	export default {
		name: "WatermarkCamera",
		props: {
			//水印文本
			markText: {
				type: String,
				default: ""
			},
			//水印位置
			markLocation: {
				type: Object,
				default: undefined
			},
			//水印填充颜色
			fillColor: {
				type: String,
				default: "#FFFFFF"
			},
			textSize: {
				type: Number,
				default: 30
			},
			//是否添加背景水印
			bgMark: {
				type: Boolean,
				default: false
			},
			//背景水印文字
			bgMarkText: {
				type: String,
				default: "水印"
			}
		},
		data() {
			return {
				tempSrc: "",
				src: "",
				canvasSiz: {
					width: "",
					height: ""
				},
			};
		},

		methods: {
			takePhoto() {
				const _this = this
				const ctx = uni.createCameraContext();
				//拍照
				ctx.takePhoto({
					quality: 'high',
					success: (res) => {
						this.tempSrc = res.tempImagePath
						//获取图片信息
						uni.getImageInfo({
							src: this.tempSrc,
							success: (image) => {
								//图片宽高
								this.canvasSiz.width = image.width
								this.canvasSiz.height = image.height

								//加个延迟，防止高宽，定位没有获取到
								setTimeout(() => {
									//图片转成canvas
									const context = uni.createCanvasContext("myCanvas",
										this)
									context.drawImage(this.tempSrc, 0, 0, image.width,
										image.height)
									context.setFillStyle(this.fillColor)
									context.setFontSize(this.textSize)
									context.fillText(this.markText, 20, image.height -
										250); //定位,可以根据传入的坐标绘制
									context.save();

									/** 添加背景水印*/
									if (this.bgMark) {
										context.translate(0, 0);
										context.rotate(30 * Math.PI / 180);
										context.setFillStyle("#bbb")
										context.setFontSize(28)
										
										/***根据需求修改为随机水印  */
										context.fillText(this.bgMarkText, 20, 20)
										context.fillText(this.bgMarkText, image.width-20, 20)
										context.fillText(this.bgMarkText, image.width/2, image.height/2)
										context.fillText(this.bgMarkText, 20, image.height - 100)
										context.fillText(this.bgMarkText, image.width-20, image.height - 100)
										/****************************/
										

										context.restore();
									}

									context.draw(false, () => {
										// 转成照片显示
										uni.canvasToTempFilePath({
												destWidth: image.width,
												destHeight: image.height,
												canvasId: 'myCanvas',
												fileType: 'png',
												success: function(res) {
													_this.src = res
														.tempFilePath
												}
											},
											_this
										);
									});
								}, 200)

							}
						})
					}
				});
			},


			savePhoto(path, name) {
				//可以在canvas的回调方法中使用，保存到相册
				uni.saveImageToPhotosAlbum({
					filePath: path,
					success: () => {
						uni.showToast({
							title: '已保存至相册',
							duration: 2000
						});
					}
				});
			},
			error(e) {
				console.log(e.detail);
			},
			completePhoto() {
				this.$emit("complete", this.src)
			},
			rePhoto() {
				this.src = ""
				this.tempSrc = ""
			},
			//返回上一层
			back() {
				setTimeout(() => {
					this.$emit("back")
				}, 100)
			}
		},

	}
</script>

<style lang="scss" scoped>
	.watermark-camera {
		.camera-box {
			position: relative;

			.back-btn {
				position: absolute;
				bottom: 130rpx;
				left: 100rpx;
			}

			.photo-outter-btn {
				position: absolute;
				width: 140rpx;
				height: 140rpx;
				border-radius: 120rpx;
				background-color: #ccc;
				bottom: 100rpx;
				left: 50%;
				margin-left: -60rpx;
				display: flex;
				align-items: center;
				justify-content: center;

				.photo-inner-btn {
					width: 100rpx;
					height: 100rpx;
					border-radius: 100rpx;
					background-color: #fff;
				}
			}
		}

		.img-box {
			width: 100%;
			height: 100vh;
			position: relative;

			.complete-photo {
				position: absolute;
				right: 20rpx;
				bottom: 50rpx;
			}

			.re-photo {
				position: absolute;
				left: 20rpx;
				bottom: 50rpx;
			}
		}
	}
</style>
