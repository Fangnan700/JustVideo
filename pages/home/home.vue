<template>
	<!-- <view class="status-bar" :style="{ height: systemInfo.statusBarHeight + 'px' }"></view> -->
	<view class="content">
		<view class="search-input-wrap" :style="{ width: systemInfo.screenWidth - 30 + 'px' }">
			<uni-easyinput class="search-input" suffixIcon="search" v-model="keyword" placeholder="请输入关键词"
				@iconClick="search" @confirm="search ">
			</uni-easyinput>
		</view>
<!-- 		<view class="type-wrap" :style="{ width: systemInfo.screenWidth - 30 + 'px' }">
			<uni-data-select :localdata="sourceType" placeholder="请选择影片类型" @change="changeType"></uni-data-select>
		</view> -->
		<view class="result-total-wrap" v-show="resultTotal != 0"
			:style="{ width: systemInfo.screenWidth - 30 + 'px' }">
			<uni-tag :text="(keyword == '' ? '资源总数' : '搜索结果') + '，共'+resultTotal+'条'"></uni-tag>
		</view>
		<view class="result-wrap">
			<uni-card v-for="(result, index) in resultList" key="index" :title="result.vod_name"
				:sub-title="result.vod_director" :extra="result.vod_class + '\n' + result.vod_year"
				:thumbnail="result.vod_pic" @click="play(result)">
				<text>{{result.vod_content.replace(/<[^>]*>/g, '')}}</text>
			</uni-card>
		</view>
	</view>
	<view class="pagination-wrap" v-show="resultTotal != 0">
		<uni-pagination :total="resultTotal" pageSize="20" prev-text="上一页" next-text="下一页" @change="changePage" />
	</view>
	<view class="footer-wrap"></view>
</template>
<script>
	export default {
		data() {
			return {
				systemInfo: null,
				sourceApi: '',
				sourceType: [],
				ids: '',
				type: 0,
				keyword: '',
				resultList: [],
				resultTotal: 0,
				page: 0,
			}
		},
		created() {
			plus.navigator.setFullscreen(false);
			this.systemInfo = uni.getSystemInfoSync();
			this.sourceApi = getApp().globalData.sourceApi;
			this.sourceType = getApp().globalData.sourceType;
		},
		onLoad() {
			this.search(1);
			setInterval(() => {
				this.systemInfo = uni.getSystemInfoSync();
			}, 500);
		},
		methods: {
			search(pg) {
				this.ids = '';
				this.resultTotal = 0;
				this.pageCount = 0;
				this.resultList = [];

				uni.request({
					url: this.sourceApi + '?ac=list&wd=' + this.keyword + '&pg=' + pg,
					success: (res) => {
						// console.log(res.data);

						this.resultTotal = res.data.total;
						this.page = res.data.page;

						let tempIds = res.data.list;
						for (let item of tempIds) {
							this.ids = this.ids + item.vod_id + ',';
						}

						if (this.ids !== '') {
							uni.request({
								url: this.sourceApi + '?ac=detail&ids=' + this.ids.slice(0, -1),
								success: (res) => {
									// console.log(res.data);
									this.resultList = res.data.list;
								},
							})
						}
					},
				});
			},
			play(rawData) {
				let reg = /(https?|http|ftp|file):\/\/[-A-Za-z0-9+&@#/%?=~_|!:,.;]+(m3u8)/g;
				let realUrl = rawData.vod_play_url.match(reg);
				let playList = [];

				realUrl.forEach((_url, _index) => {
					playList.push({
						'text': rawData.vod_name + ' 第' + (_index + 1) + '集',
						'value': _index,
						'episode': '第' + (_index + 1) + '集',
						'url': _url,
					});
				});

				// console.log(playList);
				uni.navigateTo({
					url: '/pages/player/player',
					success: (res) => {
						res.eventChannel.emit('acceptDataFromOpenerPage', {
							data: {
								'title': rawData.vod_name,
								'class': rawData.vod_class,
								'director': rawData.vod_director,
								'year': rawData.vod_year,
								'content': rawData.vod_content,
								'poster': rawData.vod_pic,
								'playList': playList
							}
						})
					}
				})
			},
			changePage(e) {
				// console.log(e)
				this.page = e.current;
				this.search(this.page);
			},
			changeType(tag) {
				// for(let t of this.sourceType) {
				// 	if(t.value == tag) {
				// 		console.log(t);
				// 	}
				// }
				
				this.type = tag;
				this.search(1);
			},
			sortByKey(array, key) {
				return array.sort(function(a, b) {
					let x = a[key];
					let y = b[key];
					return x < y ? -1 : x > y ? 1 : 0;
				});
			},
		}
	}
</script>

<style>
	.search-input-wrap {
		margin: 0 auto;
	}
	
	.type-wrap {
		margin: 10px auto;
	}

	.result-total-wrap {
		margin: 10px auto;
	}

	.pagination-wrap {
		width: fit-content;
		margin: 20px auto 10px auto;
	}

	.footer-wrap {
		width: 100%;
		height: 30px;
		margin-top: 30px;
	}
</style>