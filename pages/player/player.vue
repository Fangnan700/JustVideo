<template>
	<!-- <view class="status-bar" :style="{ height: systemInfo.statusBarHeight + 'px' }"></view> -->
	<view class="content">
		<view class="my-player-wrap">
			<video :title="playingInfo.title" :src="playingInfo.src" @error="videoErrorCallback" play-strategy='2'
				autoplay='true' vslide-gesture-in-fullscreen=true controls=true :style="{ width: systemInfo.screenWidth + 'px' }">
			</video>
		</view>
		<view class="info-wrap">
			<view>
				<uni-tag text="选集"></uni-tag>
			</view>
			<view class="episode-wrap">
				<uni-data-select :localdata="playList" @change="changeVideo"></uni-data-select>
			</view>
			<br />
			<view>
				<uni-card :title="title" :cover="poster" :sub-title="director" :extra="vod_class + '\n' + year" margin=0>
					{{constent.replace(/<[^>]*>/g, '')}}
				</uni-card>
			</view>
		</view>
		<view class="footer-wrap"></view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				systemInfo: null,
				ready: false,
				title: '',
				vod_class: '',
				director: '',
				constent: '',
				year: '',
				poster: '',
				playList: [],
				playingInfo: {},
			}
		},
		created() {
			this.systemInfo = uni.getSystemInfoSync();
		},
		onLoad(option) {
			const eventChannel = this.getOpenerEventChannel();
			eventChannel.on('acceptDataFromOpenerPage', (res) => {
				this.title = res.data.title;
				this.vod_class = res.data.class;
				this.director = res.data.director;
				this.year = res.data.year;
				this.constent = res.data.content;
				this.poster = res.data.poster;
				this.playList = res.data.playList;

				// console.log(this.playList);

				this.playingInfo = {
					'title': this.playList[0].text,
					'src': this.playList[0].url.replace('#', ''),
				}
				
				// console.log(this.playingInfo);

				this.ready = true;
			});

			setInterval(() => {
				this.systemInfo = uni.getSystemInfoSync();
			}, 500);
		},
		methods: {
			videoErrorCallback: function(e) {
				if (this.ready) {
					uni.showModal({
						content: '视频加载错误 ',
						showCancel: false
					});
					
					// console.log(e);
				}
			},
			changeVideo(tag) {
				// console.log(tag);
				const video = this.playList[tag];
				this.playingInfo = {
					'title': video.text,
					'src': video.url.replace('#', ''),
				}
			}
		}
	}
</script>

<style>
	.content {
		width: 100%;
		height: 100%;
	}

	.my-player-wrap {
		width: 100%;
	}

	.info-wrap {
		padding: 10px;
	}

	.episode-wrap {
		margin: 10px 0;
	}
	
	.footer-wrap {
		width: 100%;
		height: 30px;
		margin-top: 30px;
	}
</style>