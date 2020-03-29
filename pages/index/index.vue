<template>
	<view class="app">
		<div class="pic-container">
			<view class="picture">
				<image :src="imageUrl" mode="widthFix"></image>
				<view class="result-box" v-if="detectfaceResult">
					<view class="info">
						<view class="info-title">性别</view>
						<view class="info-detail">{{detectfaceResult.gender > 50 ? '男' : '女'}}</view>
					</view>
					<view class="info" v-if="detectfaceResult.age">
						<view class="info-title">年龄</view>
						<view class="info-detail">{{detectfaceResult.age}}</view>
					</view>
					<view class="info" v-if="detectfaceResult.beauty">
						<view class="info-title">颜值</view>
						<view class="info-detail">{{detectfaceResult.beauty}}</view>
					</view>
				</view>
			</view>
			<view class="tip">
				<icon type="info" size="16"></icon>
				<text>快来比拼颜值吧</text>
			</view>
		</div>
		<view class="operate-btns">
			<button class="share" type="primary">分享</button>
			<button class="upload" type="default" @click="uploadPicture">上传照片</button>
		</view>
		<button class="create-poster-btn" @click="createPoster">生成海报</button>
		<!-- <CreatePoster></CreatePoster> -->
		<div class="poster" @click="isShowModal=false" v-if="isShowModal">
			<div class="canvas-container">
				<canvas canvas-id="poster" @error="canvasIdErrorCallback"></canvas>
			</div>
		</div>
	</view>
</template>

<script>
	import CreatePoster from '../../components/CreatePoster.vue'
	export default {
		components: {
			CreatePoster
		},
		data() {
			return {
				imageUrl: require('../../static/images/9.png'),
				detectfaceResult: null,
				isShowModal: false,
				canDownloadImg: true
			}
		},
		onReady() {},
		methods: {
			canvasIdErrorCallback: function(e) {
				console.error(e.detail.errMsg)
			},
			async createCanvas() {
				const imgWidth = 300,
					imgHeight = 350;
				// 评分区宽高、位置
				const infowidth = 100,
					infoHeight = 120,
					infoPositionLeft = 20,
					infoPositionTop = imgHeight - 20 - infoHeight,
					infoColor = 'rgba(0,0,0,0.1)';

				let ctx = uni.createCanvasContext('poster');
				let ctxImg = await this.downloadImage(this.imageUrl);

				if (ctxImg.tempFilePath) {
					ctx.drawImage(ctxImg.tempFilePath, 0, 0, imgWidth, imgHeight);
					ctx.beginPath();
					ctx.rect(infoPositionLeft, infoPositionTop, infowidth, infoHeight, 10)
					ctx.setFillStyle(infoColor)
					ctx.setStrokeStyle(infoColor)
					ctx.fill()
					ctx.setFontSize(16)
					// 蓝色 #409eff
					ctx.fillStyle = '#fff'
					ctx.fillText(`性别：${this.detectfaceResult.gender > 50 ? '男' : '女'}`, 30, infoPositionTop + 40)
					ctx.fillText(`年龄：${this.detectfaceResult.age}`, 30, infoPositionTop + 70)
					ctx.fillText('颜值：', 30, infoPositionTop + 100)
					ctx.fillStyle = '#409eff'
					ctx.setFontSize(22)
					ctx.fillText(this.detectfaceResult.beauty, 76, infoPositionTop + 100)
					ctx.stroke()
					ctx.draw(false, () => {
						uni.canvasToTempFilePath({
							canvasId: 'poster',
							success(res) {
								uni.saveImageToPhotosAlbum({
									filePath: res.tempFilePath,
									success(r) {
										uni.showToast({
											title: '图片保存成功'
										})
									},
									fail(r) {
										uni.showToast({
											title: '图片保存失败',
											icon: 'none'
										})
									}
								})
							}
						})
					})
				}

			},
			downloadImage(url) {
				return new Promise((resolve, reject) => {
					uni.downloadFile({
						url: url,
						success: (res) => {
							return resolve(res)
						},
						fail: (err) => {
							return reject(err)
						}
					})
				})
			},
			createPoster() {
				this.isShowModal = true
				this.createCanvas()
			},
			uploadPicture() {
				uni.chooseImage({
					success: (chooseImageRes) => {
						const tempFilePaths = chooseImageRes.tempFilePaths;
						this.imageUrl = tempFilePaths[0];
						uni.showLoading({
							title: '颜值鉴定中',
							mask: true
						})
						uni.uploadFile({
							url: 'https://ai.qq.com/cgi-bin/appdemo_detectface',
							filePath: tempFilePaths[0],
							name: 'image_file',
							success: res => {
								// this.canDownloadImg = true;
								const result = JSON.parse(res.data)
								if (result.ret === 0) {
									// 成功获取分析结果
									this.detectfaceResult = result.data.face[0];
								} else {
									// 检测失败
									uni.showToast({
										icon: 'none',
										title: '好像没找到你的小脸哦~'
									})
								}
								uni.hideLoading()
							}
						});
					}
				})
			}
		}
	}
</script>

<style lang="less">
	@white: #fff;
	@Color1: #409eff;

	.app {
		width: 750rpx;

		.pic-container {
			background-color: #f6f6f6;
			padding-top: 20rpx;

			.picture {
				width: 690rpx;
				border: 1px solid #ccc;
				margin: 0 auto;
				border-radius: 16rpx;
				background-color: @white;
				display: flex;
				justify-content: center;
				align-items: center;
				position: relative;

				image {
					width: 100%;
					border-radius: 16rpx;
				}

				.result-box {
					position: absolute;
					bottom: 20rpx;
					left: 20rpx;
					padding: 20rpx;
					background: rgba(0, 0, 0, 0.06);

					.info {
						display: flex;
						align-items: center;
						padding: 0 5rpx;
						font-size: 28rpx;
						color: @white;
						margin-bottom: 10rpx;
					}

					.info-title {
						margin-right: 20rpx;
					}

					.info-detail {
						padding: 5rpx 20rpx;
						color: @white;
						background: @Color1;
						border-radius: 10rpx;
					}
				}
			}

			.tip {
				height: 80rpx;
				display: flex;
				align-items: center;

				icon {
					margin: 20rpx 30rpx;
				}

				text {
					font-size: 28rpx;
				}
			}
		}

		.operate-btns {
			display: flex;
			justify-content: space-around;
			margin-top: 40rpx;

			button {
				width: 240rpx;

				&.share {
					background: #409eff;
					border: 1px solid @white;
					color: @white;
				}

				&.upload {
					background: @white;
					border: 1px solid #409eff;
					color: #409eff;
				}
			}
		}

		.poster {
			position: fixed;
			top: 0;
			right: 0;
			bottom: 0;
			left: 0;
			background-color: rgba(0, 0, 0, 0.3);
			display: flex;
			justify-content: center;
			align-items: center;

			.canvas-container {
				width: 600rpx;
				height: 700rpx;
				background-color: #fff;
				border: 1px solid #fff;

				canvas {
					width: 600rpx;
					height: 700rpx;
				}
			}
		}

		.create-poster-btn {
			width: 240rpx;
			margin-top: 100rpx;
		}
	}
</style>
