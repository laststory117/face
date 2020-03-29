<template>
	<view class="create-poster" v-if="isShow">
		<view class="container">
			<canvas canvas-id="poster"></canvas>
		</view>
	</view>
</template>
<script>
	export default {
		name: 'CreatePoster',
		props: {
			isShow: {
				type: Boolean,
				requireed: true,
				default: false
			}
		},
		methods: {
			createCanvas() {
				let ctx = uni.createCanvasContext('poster');
				let img = new Image();
				// img.src = this.
			}
		},
		onLoad() {
			this.createCanvas()
		}
	}
</script>
<style lang="less">
	.create-poster {
		position: fixed;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		background-color: rgba(0, 0, 0, 0.3);
		display: flex;
		justify-content: center;
		align-items: center;
		.container {
			width: 600rpx;
			height: 600rpx;
			background-color: #FFFFFF;
			border-radius: 20rpx;
		}
	}	
</style>