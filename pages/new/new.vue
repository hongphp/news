<template>
	<view class="index">
		<block v-for="(item, index) in list" :key="index">
			<view class="card">
				<view class="car-title-view" v-if="item.mblog.text" v-html="item.mblog.text">
					
				</view>
				<image
					class="card-img"
					v-if="item.mblog.original_pic"
					:src="item.mblog.original_pic"
					mode="widthFix"
					@click="previmg(item.mblog.original_pic)"
				></image>
				<video  class="video" v-if="item.mblog.page_info"
					:src="item.mblog.page_info.media_info.stream_url">
					
				</video>
				
				
			</view>
		</block>
		<text class="loadMore" v-if="loading">加载中...</text>
		<button
			open-type="getUserInfo"
			v-if="showlogin"
			id="showlogin"
			type="warn"
			@getuserinfo="login"
		>
			使用微信登录
		</button>
		<view id="mask" @click="showlogin = false" v-if="showlogin"></view>
	</view>
</template>
<script>
/**
 * 首页
 */
import util, {checklogin } from '../../common/util';
export default {
	data() {
		return {
			/* 为true表示正在刷新 */
			refreshing: false,
			/* 服务商列表 */
			providerList: [],
			/* 数据数组 */
			list: [],
			openid: '',
			userinfo: '',
			token: '',
			/* 当前页数 */
			fetchPageNum: 1,
			/* 为true表示加载中 */
			loading: true,
			showlogin: false
		};
	},
	mounted: function() {
		//#ifdef APP-PLUS
		plus.navigator.closeSplashscreen();
		//#endif
	},
	onShow() {
		this.checkstat();
	},
	onLoad() {
		this.getData()
		
	},
	onReachBottom() {
		/* 到底部加载 */
		
	},
	onPullDownRefresh() {
		// 下拉刷新
		this.refreshing = true;
		
	},
	onShareAppMessage() {
		return {
			title: '最新最好笑的趣事趣图，来看下吧',
			path: '/pages/new/new'
		};
	},
	methods: {
		checkstat() {
			var logininfo = util.checklogin();
			this.openid = logininfo.openid;
			this.userinfo = logininfo.userinfo;
			this.token = logininfo.token;
		},
		goto(url) {
			location.assign(url)
		},
		getData() {
			this.loading = true;
			var self = this;
			uni.request({
				url: util.apiurl + 'api/v1/hot_list',
				method: 'get',
				dataType: 'json',
				data: {
					
				},
				success: function(ret) {
					
					if (ret.statusCode !== 200) {
						uni.showToast({
							title: '请求数据失败请重试' + ret.statusCode,
							duration: 2000,
							icon: 'none'
						});
					} else {
						if (ret.data.length < 1) {
							uni.showToast({
								title: '没有了',
								icon: 'none'
							});
							return false;
						}
			
						if (self.refreshing) {
							self.refreshing = false;
							uni.stopPullDownRefresh();
							self.list = ret.data;
						
							self.fetchPageNum = 2;
						} else {
							console.log(ret.data)
							
							self.list = self.list.concat(ret.data);
							self.fetchPageNum += 1;
							
						}
						for (let i = 0; i < self.list.length; i++) {
								
										self.list[i].mblog.page_info = false
									
								}
								
					}
				},
				fail: res => {
					
					uni.showToast({
						title: '请求数据失败请重试1',
						duration: 2000,
						icon: 'none'
					});
				},
				complete: function() {
					this.loading = false;
				},
				header: {
					Authorization: util.httpauth
				}
			});
		},
		showreply(id, index) {
			uni.setStorage({
				key: 'detail-' + id,
				data: this.list
			});
			// 进入回复页面
			uni.navigateTo({
				url: '../detail/detail?id=' + id
			});
		},
		setlike(type, index, tid) {
			util.setlike(this, this.list[index], type, tid);
		},
		previmg(url) {
			uni.previewImage({
				urls: [url]
			});
		},
		/* 登录方法 */
		login(res) {
			util.logincomm(res, this);
		}
	}
};
</script>

<style>
	.car-title-view {
		padding: 60upx
	}
	.video {
		margin-left: 50upx;
	}
</style>
