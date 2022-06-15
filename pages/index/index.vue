<template>
	<view class="index">
		<musichead title="网易云音乐" :icon="false"></musichead>
		<view class="container">
			<scroll-view scroll-y="true">
				<!-- 搜索区域 -->
				<view class="index-search">
					<text class="iconfont icon-search"></text>
					<input type="text" placeholder="搜索歌曲" @tap="handleToSearch()" />
				</view>
				<view v-if="isLoading">
					<m-for-skeleton :avatarSize="200" :row="3" :loading="isLoading" :isarc="'square'" :titleStyle="{}" :title="false"
						v-for="(item,key) in 4" :key="key">
					</m-for-skeleton>
				</view>
				<view class="index-list" v-else>
					<view class="index-list-item" v-for="(item, index) in topsList" :key="index"
						@tap="handleToList(item.id)">
						<view class="index-list-img">
							<image :src="item.coverImgUrl" mode=""></image>
							<text>{{ item.updateFrequency }}</text>
						</view>
						<view class="index-list-text">
							<view v-for="(it, ind) in item.tracks" :key="ind">{{ ind + 1 }}.{{ it.first }} -
								{{ it.second }}
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
	} from "../../common/config.js";
	import mForSkeleton from "@/components/m-for-skeleton/m-for-skeleton";
	export default {
		data() {
			return {
				topsList: [],
				isLoading: true
			};
		},
		components: {
			musichead,
			mForSkeleton
		},
		onLoad() {
			this.topList().then((res) => {
				if(res.length){
					setTimeout(()=>{
						this.topsList = res;
						// console.log(this.topsList);
						this.isLoading =false
					},1000)
				}
			});
		},
		methods: {
			topList() {
				return new Promise(function(reslove, reject) {
					uni.request({
						url: `${baseUrl}/toplist/detail`,
						method: "GET",
						data: {},
						success: (res) => {
							console.log(res);
							let result;
							result = res.data.list;
							result.length = 4;
							reslove(result);
						}
					});
				});
			},
			handleToList(id) {
				uni.navigateTo({
					url: '/pages/list/list?id=' + id
				})
			},
			handleToSearch() {
				uni.navigateTo({
					url: '/pages/search/search'
				})
			}
		},
	};
</script>

<style scoped>
	.index {}

	.index-search {
		display: flex;
		align-items: center;
		height: 70rpx;
		margin: 70rpx, 30rpx, 30rpx, 30rpx;
		background: #f7f7f7;
		border-radius: 50rpx;
	}

	.index-search text {
		font-size: 26rpx;
		margin-right: 26rpx;
		margin-left: 28rpx;
	}

	.index-search input {
		font-size: 28rpx;
		flex: 1;
	}

	.index-list {
		margin: 0, 30rpx;
	}

	.index-list-item {
		display: flex;
		margin-bottom: 34rpx;
	}

	.index-list-img {
		width: 212rpx;
		height: 212rpx;
		position: relative;
		border-radius: 30rpx;
		overflow: hidden;
		margin-right: 22rpx;
	}

	.index-list-img image {
		width: 100%;
		height: 100%;
	}

	.index-list-img text {
		position: absolute;
		left: 12rpx;
		bottom: 16rpx;
		font-size: 20rpx;
		color: white;
	}

	.index-list-text {
		font-size: 24rpx;
		line-height: 66rpx;
	}
</style>
