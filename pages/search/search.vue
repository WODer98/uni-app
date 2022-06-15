<template>
	<view class="search">
		<musichead title="网易云音乐" :icon="true" :iconblack="true"></musichead>
		<!-- 搜索区域 -->
		<view class="container">
			<scroll-view scroll-y="true">
				<view class="search-search">
					<text class="iconfont icon-search"></text>
					<input type="text" placeholder="搜索歌曲" v-model="searchWord" @confirm="handleToSearch(searchWord)"
						@input="handleToSuggest" />
					<text v-show="searchType !=1" class="iconfont icon-guanbi" @tap="handleToDelete()"></text>
				</view>
				<block v-if="searchType == 1">
					<view class="search-history">
						<view class="search-history-head">
							<text>历史记录</text>
							<text class="iconfont icon-lajitong" @tap="handleToClean()"></text>
						</view>
						<view class="search-history-list">
							<view v-for="(item,index) in searchHistory" :key="index" @tap="handleToWord(item)">{{item}}
							</view>
						</view>
					</view>
					<view class="search-hot">
						<view class="search-hot-head">
							热搜榜
						</view>
						<view class="search-hot-item" v-for="(item,index) in searchHots" :key="index"
							@tap="handleToWord(item.searchWord)">
							<view class="search-hot-top">{{index+1}}</view>
							<view class="search-hot-word">
								<view>
									{{item.searchWord}}
									<image :src="item.iconUrl" mode="aspectFit"></image>
								</view>
								<view>{{item.content}}</view>
							</view>
							<text class="search-hot-count">{{item.score}}</text>
						</view>
					</view>
				</block>
				<block v-else-if="searchType == 2">
					<view class="search-result">
						<view class="search-result-item" v-for="(item,index) in searchList" :key="index"
							@tap="handleToDetail(item.id)">
							<view class="search-result-word">
								<view>{{item.name}}</view>
								<view>{{item.artists[0].name}}-{{item.album.name}}</view>
							</view>
							<text class="iconfont icon-bofang_o"></text>
						</view>
					</view>
				</block>
				<block v-else-if="searchType == 3">
					<view class="search-suggest">
						<view class="search-suggest-head" @tap="handleToWord(searchWord)">搜索“{{searchWord}}”</view>
						<view class="search-suggest-item" v-for="(item,index) in searchSuggestList" :key="index" @tap="handleToWord(item.keyword)">
							<text class="iconfont icon-search"></text>{{item.keyword}}
						</view>
					</view>
				</block>
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
	export default {
		comments: {
			musichead
		},
		data() {
			return {
				searchHots: [],
				searchWord: '',
				searchHistory: [],
				searchType: 1,
				searchList: [],
				searchSuggestList: []
			}
		},
		methods: {
			searchHot() {
				return uni.request({
					url: `${baseUrl}/search/hot/detail`,
					method: "GET",
				})
			},

			searchWordLink(word) {
				return uni.request({
					url: `${baseUrl}/search?keywords=${word}`,
					method: 'GET'
				})
			},

			searchSuggest(word) {
				return uni.request({
					url: `${baseUrl}/search/suggest?keywords=${word}&type=mobile`,
					method: 'GET'
				})
			},

			handleToWord(word) {
				this.searchWord = word
				this.handleToSearch(word)
			},

			handleToSearch(word) {
				this.searchHistory.unshift(word)
				this.searchHistory = [...new Set(this.searchHistory)]
				if (this.searchHistory.length > 10) {
					this.searchHistory.length = 10
				}
				uni.setStorage({
					key: 'searchHistory',
					data: this.searchHistory

				})
				this.getSearchList(word)
				this.searchType = 2
			},
			handleToClean() {
				uni.removeStorage({
					key: 'searchHistory',
					success: (res) => {
						this.searchHistory = []
					}
				})
			},
			getSearchList(word) {
				this.searchWordLink(word).then((res) => {
					if (res[1].data.code == '200') {
						// console.log(res);
						this.searchList = res[1].data.result.songs
						// console.log(this.searchList);
					}
				})
			},
			handleToDelete() {
				this.searchWord = ''
				this.searchType = 1
			},
			handleToDetail(songId) {
				// console.log(songId);
				uni.navigateTo({
					url: '/pages/detail/detail?songId=' + songId,
				});
			},
			handleToSuggest(ev) {
				let value = ev.detail.value
				if (!value) {
					this.searchType = 1;
					return;
				}
				this.searchSuggest(value).then((res) => {
					if (res[1].data.code == '200') {
						this.searchSuggestList = res[1].data.result.allMatch
						// console.log(this.searchSuggestList);
						this.searchType = 3
					}
				})
			}
		},
		onLoad() {
			this.searchHot().then((res) => {
				// console.log(res[1].data.data);
				if (res[1].data.code == '200') {
					this.searchHots = res[1].data.data;
				}
			})
			uni.getStorage({
				key: 'searchHistory',
				success: (res) => {
					this.searchHistory = res.data
				}
			})
		}
	}
</script>

<style scoped>
	.search-search {
		display: flex;
		align-items: center;
		height: 70rpx;
		margin: 70rpx, 30rpx, 50rpx, 30rpx;
		background: #f7f7f7;
		border-radius: 50rpx;
		margin-bottom: 20rpx;
	}

	.search-search text {
		margin: 0 26rpx;
	}

	.search-search input {
		flex: 1;
		font-size: 26rpx;
	}

	.search-history {
		margin: 0 30rpx 50rpx 30rpx;
		font-size: 26rpx;
	}

	.search-history-head {
		display: flex;
		justify-content: space-between;
		margin-bottom: 26rpx;
	}

	.search-history-list {
		display: flex;
		flex-flow: wrap;
	}

	.search-history-list view {
		padding: 16rpx 28rpx;
		border-radius: 40rpx;
		margin-right: 30rpx;
		margin-bottom: 30rpx;
		background: #F7F7F7;
	}

	.search-hot {
		margin: 0 30rpx;
		font-size: 26rpx;
	}

	.search-hot-head {
		margin-bottom: 36rpx;
	}

	.search-hot-item {
		display: flex;
		align-items: center;
		margin-bottom: 58rpx;
	}

	.search-hot-top {
		color: #fb2222;
		width: 60rpx;
		margin-left: 8rpx;
	}

	.search-hot-word {
		flex: 1;
		font-size: 30rpx;
	}

	.search-hot-word view:nth-child(1) {
		font-size: 30rpx;
		color: black;
	}

	.search-hot-word view:nth-child(2) {
		font-size: 24rpx;
		color: #878787;
	}

	.search-hot-word image {
		width: 48rpx;
		height: 22rpx;
	}

	.search-hot-count {
		color: #878787;
	}

	.search-result {
		border-top: 2rpx #e4e4e4 solid;
		padding: 30rpx;
	}

	.search-result-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding-bottom: 30rpx;
		margin-bottom: 30rpx;
		border-bottom: 2rpx #e4e4e4 solid;
	}

	.search-result-word {}

	.search-result-word view:nth-child(1) {
		font-size: 28rpx;
		color: #275790;
		margin-bottom: 12rpx;
	}

	.search-result-word view:nth-child(2) {
		font-size: 22rpx;
		color: #898989;
	}

	.search-result-item text {
		font-size: 28rpx;
	}

	.search-suggest {
		border-top: 2rpx #e4e4e4 solid;
		padding: 30rpx;
		font-size: 26rpx;

	}

	.search-suggest-head {
		color: #4574a5;
		margin-bottom: 74rpx;
	}

	.search-suggest-item {
		color: #5d5d5d;
		margin-bottom: 74rpx;
	}

	.search-suggest-item text {
		color: #bdbdbd;
		margin-right: 28rpx;
		position: relative;
		top: 2rpx;
	}
</style>
