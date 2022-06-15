<template>
	<view class="list">
		<view class="fixbg" :style="{'background-Image': 'url('+playList.coverImgUrl+')'}"></view>
		<musichead title="歌单" :icon="true" color="white"></musichead>
		<view class="container" v-show="!isLoading">
			<scroll-view scroll-y="true">
				<!-- 歌单头部 -->
				<view class="list-head">
					<view class="list-head-img">
						<image :src="playList.coverImgUrl" mode=""></image>
						<text class="iconfont icon-yousanjiao">{{playList.playCount| formatCount}}</text>
					</view>
					<view class="list-head-text">
						<view>{{playList.name}}</view>
						<view>
							<image :src="playList.creator.avatarUrl" mode=""></image>
							{{playList.creator.nickname}}
						</view>
						<view>
							{{playList.description}}
						</view>
					</view>
				</view>
				<!-- 条件编译：符合条件的才显示出来 -->
				<!-- #ifdef MP-WEIXIN -->
				<button class="list-share" open-type="share">
					<text class="iconfont icon-fenxiang"> 分享给微信好友</text>
				</button>
				<!-- #endif -->
				<!-- 歌曲列表 -->
				<view class="list-music">
					<view class="list-music-head">
						<text class="iconfont icon-bofang"></text>
						<text>播放全部</text>
						<text>(共{{playList.trackCount}}首)</text>
					</view>
					<view class="list-music-item" v-for="(item,index) in playList.tracks" :key="index" @tap="handleToDetail(item.id)">
						<view class="list-music-top">{{index + 1}}</view>
						<view class="list-music-song">
							<view>{{item.name}}</view>
							<view>
								<image v-if="privileges[index].flag > 60 && privileges[index].flag < 70" src="../../static/dujia.png" mode=""></image>
								<image v-if="privileges[index].maxbr == 999000" src="../../static/sq.png" mode=""></image>
								{{item.ar[0].name}} - {{item.name}}
							</view>
						</view>
						<text class="iconfont icon-bofang_o"></text>
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
		data() {
			return {
				playList: {
					coverImgUrl: '',
					creator: {
						avatarUrl: ''
					},
					trackCount: '',
				},
				privileges: '',
				isLoading: true,
			}
		},
		comments: {
			musichead
		},
		methods: {
			getMusicList(id) {
				return uni.request({
					url: `${baseUrl}/playlist/detail?id=${id}`,
					method: "GET",
				})
			},
			handleToDetail(songId){
				uni.navigateTo({
					url: '/pages/detail/detail?songId='+songId
				});
			}
		},
		onLoad(options) {
			uni.showLoading({
				title:'加载中...'
			});
			// console.log(options.id);
			this.getMusicList(options.id).then((res) => {
				// console.log(res);
				if (res[1].data.code == '200') {
					this.playList = res[1].data.playlist;
					this.privileges = res[1].data.privileges;
					this.$store.commit('INIT_TOPLISTIDS',res[1].data.playlist.trackIds)
					this.isLoading = false;
					uni.hideLoading();
				}
			})
		}
	}
</script>

<style scoped>
	.list-head {
		display: flex;
		margin: 30rpx;
	}

	.list-head-img {
		width: 264rpx;
		height: 264rpx;
		border-radius: 30rpx;
		overflow: hidden;
		position: relative;
		margin-right: 42rpx;
	}

	.list-head-img image {
		width: 100%;
		height: 100%;
	}

	.list-head-img text {
		position: absolute;
		right: 8rpx;
		top: 8rpx;
		color: white;
		font-size: 26rpx;
	}

	.list-head-text {
		flex: 1;
		color: #f0f2f7;
	}

	.list-head-text view:nth-child(1) {
		color: white;
		font-size: 34rpx;
	}

	.list-head-text view:nth-child(2) {
		display: flex;
		margin: 20rpx 0;
	}

	.list-head-text view:nth-child(2) image {
		width: 54rpx;
		height: 54rpx;
		border-radius: 50%;
		margin-right: 14rpx;
		font-size: 24rpx;
		align-items: center;
	}

	.list-head-text view:nth-child(3) {
		line-height: 34rpx;
		font-size: 22rpx;
	}

	.list-share {
		width: 330rpx;
		height: 74rpx;
		margin: 0 auto;
		background: rgb(0, 0, 0, 0.4);
		border-radius: 37rpx;
		color: white;
		font-size: 28rpx;
		text-align: center;
		line-height: 74rpx;
	}

	.list-share text {
		margin-right: 16rpx;
	}

	.list-music {
		background: white;
		border-radius: 50rpx;
		margin-top: 40rpx;
		overflow: hidden;
	}

	.list-music-head {
		height: 50rpx;
		margin: 30rpx 0 70rpx 22rpx;
	}

	.list-music-head text:nth-child(1) {
		height: 50rpx;
		font-size: 50rpx;

	}

	.list-music-head text:nth-child(2) {
		font-size: 30rpx;
		margin: 0 10rpx 0 26rpx;
	}

	.list-music-head text:nth-child(3) {
		font-size: 26rpx;
		color: #b2b2b2b2;
	}

	.list-music-item {
		display: flex;
		margin: 0 32rpx 60rpx 46rpx;
		align-items: center;
		color: #959595;
	}

	.list-music-top {
		width: 58rpx;
		font-size: 30rpx;
		line-height: 50rpx;
	}

	.list-music-song {
		flex: 1;
	}
	/* 歌曲名 */
	.list-music-song view:nth-child(1) {
		font-size: 28rpx;
		color: black;
		width: 70vw;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}
	/* 歌手-歌名 */
	.list-music-song view:nth-child(2) {
		/* display: flex; */
		font-size: 28rpx;
		align-items: center;
		width: 70vw;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}
	/* 独家 SQ */
	.list-music-song view:nth-child(2) image {
		width: 32rpx;
		height: 20rpx;
		margin-right: 10rpx;
		flex-shrink: 0;
	}

	.list-music-item text {
		font-size: 50rpx;
		color: #c7c7c7;
	}
</style>
