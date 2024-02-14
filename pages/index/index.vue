<template>
	<view class="index-view">
		<image class="index-img" src="../../static/huh.gif" mode="scaleToFill"></image>
	</view>
</template>

<script>
	export default {
		created() {
			/* #ifdef APP-PLUS */
			// plus.navigator.setStatusBarStyle("light");
			plus.navigator.setFullscreen(true);
			/* #endif */
		},
		mounted() {
			setTimeout(() => {
				uni.navigateTo({
					url: '/pages/home/home'
				})
			}, 3000);
		},
	}
</script>

<style>
	.index-view {
		width: 100%;
		height: 100%;
	}

	.index-img {
		width: 100%;
		height: 100%;
	}
</style>