<template>
	<view class="detail">
		<view class="fixbg" :style="{'background-Image': 'url('+songDetail.al.picUrl+')'}"></view>
		<musichead :title="songDetail.name" :icon="true" color="white"></musichead>
		<view class="container" v-show="!isLoading">
			<scroll-view scroll-y="true">
				<view class="detail-play" @tap="handleToPlay()">
					<image :src="songDetail.al.picUrl" :class="{'detail-play-run' : isPlay}" mode=""></image>
					<text class="iconfont" :class="iconPlay"></text>
					<view></view>
				</view>
				<view class="detail-lyric">
					<view class="detail-lyric-wrap" :style="{transform:'translateY('+ -(lyricIndex-1)*82+'rpx)'}">
						<view class="detail-lyric-item" :class="{active: lyricIndex == index}"
							v-for="(item,index) in songLyric" :key="index">{{item.lyric}}</view>

					</view>
				</view>
				<view class="detail-like">
					<view class="detail-like-head">喜欢这首歌的人也听</view>
					<view v-for="(item,index) in songSimi" class="detail-like-item" @tap="handleToSimi(item.id)">
						<view class="detail-like-img">
							<image :src="item.album.blurPicUrl" mode=""></image>
						</view>
						<view class="detail-like-song">
							<view>{{item.name}}</view>
							<view>
								<image v-if="item.privilege.flag >60 && item.privilege.flag<70"
									src="../../static/dujia.png" mode=""></image>
								<image v-if="item.privilege.maxbr == 999000" src="../../static/sq.png" mode=""></image>
								{{item.album.artists[0].name}}-{{item.name}}
							</view>
						</view>
						<text class="iconfont icon-bofang_o"></text>
					</view>
				</view>
				<view class="detail-like-head">精彩评论</view>
				<view class="detail-comment" v-for="(item,index) in songComment" :key="index">
					<view class="detail-comment-head"></view>
					<view class="detail-comment-item">
						<view class="detail-comment-img">
							<image :src="item.user.avatarUrl" mode=""></image>
						</view>
						<view class="detail-comment-content">
							<view class="detail-comment-title">
								<view class="detail-comment-name">
									<view>{{item.user.nickname}}</view>
									<view>{{item.timeStr}}</view>
								</view>
								<view class="detail-comment-like">{{item.likedCount}} <text
										class="iconfont icon-icon"></text></view>
							</view>
							<view class="detail-comment-text">
								{{item.content}}
							</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import "@/common/iconfont.css";
	import musichead from "../../components/musichead/musichead.vue";
	import {
		baseUrl
	} from '../../common/config.js'
	export default {
		comments: {
			musichead
		},
		data() {
			return {
				songDetail: {
					al: {
						picUrl: ''
					}
				},
				songSimi: [],
				songComment: [],
				songLyric: [],
				lyricIndex: 0,
				iconPlay: 'icon-zanting',
				isPlay: true,
				isLoading: true
			}
		},
		methods: {
			/* 
			获取歌曲详细信息
			接口示例： http://localhost:3000/song/detail?ids=347230 */
			getSongDetail(songId) {
				return uni.request({
					url: `${baseUrl}/song/detail?ids=${songId}`,
					method: 'GET'
				})
			},

			/*
			获取相似歌曲信息
			接口示例:http://localhost:3000/simi/song?id=347230 */
			getSimilarSong(songId) {
				return uni.request({
					url: `${baseUrl}/simi/song?id=${songId}`,
					method: 'GET'
				})
			},

			/* 
				获取歌曲评论信息
				接口示例：http://localhost:3000/comment/music?id=186016&limit=1
			 */
			getSongComment(songId) {
				return uni.request({
					url: `${baseUrl}/comment/music?id=${songId}`,
					method: 'GET'
				})
			},

			/*
				获取歌曲歌词信息
				接口示例：http://localhost:3000/lyric?id=33894312
			 */
			getSongLyric(songId) {
				return uni.request({
					url: `${baseUrl}/lyric?id=${songId}`,
					method: 'GET'
				})
			},

			/*
				获取歌曲url
				接口示例：http://localhost:3000/song/url?id=33894312
			 */
			getSongUrl(songId) {
				return uni.request({
					url: `${baseUrl}/song/url?id=${songId}`,
					method: 'GET'
				})
			},

			getMusic(songId) {

				this.$store.commit('NEXT_ID', songId)

				uni.showLoading({
					title: "加载中..."
				});
				this.isLoading = true

				Promise.all([this.getSongDetail(songId),
					this.getSimilarSong(songId),
					this.getSongComment(songId),
					this.getSongLyric(songId),
					this.getSongUrl(songId)
				]).then((
					res) => {
					// console.log(res);
					if (res[0][1].data.code == '200') {
						this.songDetail = res[0][1].data.songs[0]
						// console.log(this.songDetail);
					}
					if (res[1][1].data.code == '200') {
						this.songSimi = res[1][1].data.songs
					}
					// console.log(res);
					if (res[2][1].data.code == '200') {
						this.songComment = res[2][1].data.hotComments
					}
					if (res[3][1].data.code = '200') {
						console.log(res[3][1].data);
						let lyric = res[3][1].data.lrc.lyric;
						let re = /\[([^\]]+)\]([^\[]+)/g
						var result = []
						lyric.replace(re, ($0, $1, $2) => {
							result.push({
								"time": this.formatTimeToSec($1),
								"lyric": $2
							})
						})
						// console.log(result);
						this.songLyric = result
					}
					if (res[4][1].data.code == '200') {

						// #ifdef MP-WEIXIN
						this.bgAudioManager = uni.getBackgroundAudioManager();
						this.bgAudioManager.title = this.songDetail.name;
						// #endif

						// #ifdef H5
						if (!this.bgAudioManager) {
							this.bgAudioManager = uni.createInnerAudioContext();
						}

						this.iconPlay = 'icon-bofang_o';
						this.isPlay = false;
						// #endif


						// console.log(res[4][1].data);
						this.bgAudioManager.src = res[4][1].data.data[0].url || ''
						this.listenLyricIndex()
						this.bgAudioManager.onPlay(() => {
							this.iconPlay = 'icon-zanting'
							this.isPlay = true
						})
						this.bgAudioManager.onPause(() => {
							this.iconPlay = 'icon-bofang_o'
							this.isPlay = false
						})
						this.bgAudioManager.onEnded(() => {
							this.getMusic(this.$store.state.nextId)
						})
					}
					this.isLoading = false
					uni.hideLoading()
				})
			},
			formatTimeToSec(value) {
				let arr = value.split(":");
				return (Number(arr[0] * 60) + Number(arr[1])).toFixed(1)
			},
			handleToPlay() {
				if (this.bgAudioManager.paused) {
					this.bgAudioManager.play()
				} else {
					this.bgAudioManager.pause()
					this.cancelLyricIndex()
				}
			},
			listenLyricIndex() {
				clearInterval(this.timer)
				this.timer = setInterval(() => {
					for (var i = 0; i < this.songLyric.length; i++) {
						if (this.bgAudioManager.currentTime > this.songLyric[this.songLyric.length - 1].time) {
							this.lyricIndex = this.songLyric.length - 1
							break
						}
						if (this.bgAudioManager.currentTime > this.songLyric[i].time && this.bgAudioManager
							.currentTime < this.songLyric[i + 1].time) {
							this.lyricIndex = i
						}
					}
					// console.log(this.lyricIndex);
				}, 500)
			},
			cancelLyricIndex() {
				clearInterval(this.timer);
			},

			handleToSimi(songId) {
				this.getMusic(songId);
			}
		},

		onLoad(options) {
			console.log(options.songId);

			this.getMusic(options.songId)

		},

		onUnload() {
			this.cancelLyricIndex()
			// #ifdef H5
			this.bgAudioManager.destroy()
			// #endif
		},
		onHide() {
			this.cancelLyricIndex()
			// #ifdef H5
			this.bgAudioManager.destroy()
			// #endif
		}
	}
</script>

<style scoped>
	.detail-play {
		width: 580rpx;
		height: 580rpx;
		background: url(../../static/disc.png);
		background-size: cover;
		margin: 214rpx auto 0 auto;
		position: relative;
	}

	.detail-play image {
		width: 370rpx;
		height: 370rpx;
		border-radius: 50%;
		position: absolute;
		left: 0;
		top: 0;
		bottom: 0;
		right: 0;
		margin: auto;
		animation: 10s linear move infinite;
		animation-play-state: paused;
	}

	.detail-play .detail-play-run {
		width: 370rpx;
		height: 370rpx;
		border-radius: 50%;
		position: absolute;
		left: 0;
		top: 0;
		bottom: 0;
		right: 0;
		margin: auto;
		animation: 10s linear move infinite;
		animation-play-state: running;
	}

	@keyframes move {
		from {
			transform: rotate(0deg);
		}

		to {
			transform: rotate(360deg);
		}
	}

	.detail-play text {
		width: 100rpx;
		height: 100rpx;
		font-size: 100rpx;
		color: white;
		position: absolute;
		left: 0;
		top: 0;
		bottom: 0;
		right: 0;
		margin: auto;
	}

	.detail-play view {
		width: 230rpx;
		height: 360rpx;
		background: url(~@/static/needle.png);
		position: absolute;
		left: 300rpx;
		right: 0;
		height: -500rpx;
		margin: auto;
		background-size: cover;
	}

	.detail-lyric {
		font-size: 32rpx;
		line-height: 82rpx;
		height: 246rpx;
		text-align: center;
		overflow: hidden;
		color: #6e6f73;
	}

	.detail-lyric-wrap {
		transition: .5s;
	}

	.detail-lyric-item {
		height: 82rpx;

	}

	.detail-lyric-item.active {
		color: white;
	}

	.detail-like {
		margin: 0 30rpx;
	}

	.detail-like-head {
		font-size: 36rpx;
		color: white;
		margin: 50rpx 0;
	}

	.detail-like-item {
		display: flex;
		align-items: center;
		margin-bottom: 32rpx;
	}

	.detail-like-item text {
		font-size: 50rpx;
		color: #c6c2bf;
	}

	.detail-like-img {
		width: 82rpx;
		height: 82rpx;
		border-radius: 20rpx;
		overflow: hidden;
		margin-right: 20rpx;
	}

	.detail-like-img image {
		width: 100%;
		height: 100%;
	}

	.detail-like-song {
		flex: 1;
		color: #c6c2bf;
	}

	.detail-like-song view:nth-child(1) {
		font-size: 28rpx;
		color: white;
		margin-bottom: 12rpx;
	}

	.detail-like-song view:nth-child(2) {
		font-size: 22rpx;
	}

	.detail-like-song image {
		width: 26rpx;
		height: 20rpx;
		margin-right: 10rpx;
	}

	.detail-comment {
		margin: 0 30rpx;
	}

	.detail-comment-head {
		font-size: 36rpx;
		color: white;
		margin: 50rpx 0;
	}

	.detail-comment-item {
		margin-bottom: 28rpx;
		display: flex;

	}

	.detail-comment-img {
		width: 64rpx;
		height: 64rpx;
		border-radius: 50%;
		overflow: hidden;
		margin: 18rpx;
	}

	.detail-comment-img image {
		width: 100%;
		height: 100%;

	}

	.detail-comment-content {
		flex: 1;
		color: #cbcacf;
	}

	.detail-comment-title {
		display: flex;
		justify-content: space-between;
	}

	.detail-comment-name {}

	.detail-comment-name view:nth-child(1) {
		font-size: 26rpx;
	}

	.detail-comment-name view:nth-child(2) {
		font-size: 20rpx;
	}

	.detail-comment-like {
		font-size: 28rpx;
	}

	.detail-comment-text {
		font-size: 28rpx;
		line-height: 40rpx;
		color: white;
		margin-top: 20rpx;
		border-bottom: 1rpx #e0e0e0 solid;
		padding-bottom: 40rpx;
	}
</style>
