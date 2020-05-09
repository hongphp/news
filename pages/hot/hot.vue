<template>
	<view class="index">
		<block v-for="(item, index) in list" :key="index">
			<view class="card">
				
				<view class="car-title-view" v-if="item.title" @click="setlike(item)">
					<rich-text :nodes="item.title">
					</rich-text>
					<image
						class="card-img"
						v-if="item.image"
						:src="item.image"
						mode="widthFix"
						@click="setlike(item)"
					></image>
					<text class="active" v-if="item.image">点击量： {{ item.hitcount }}</text>
				</view>
				<!-- <video v-if="item.video" :src="item.video" controls></video> -->
				
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
/* *
 * tabbar最热页面
 * */
import { mapState, mapMutations, mapActions } from 'vuex';
import util, { parGetData, parSetData, checklogin } from '../../common/util';
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
			showlogin: false,
			//滚动条object
			scroll: null
		};
	},
	onShow() {
		this.checkstat();
	},
	onShareAppMessage() {
		return {
			title: '最新资讯趣图，来看下吧',
			path: '/pages/hot/hot'
		};
	},
	onLoad() {
		this.checkstat();
		this.getData();
	},
	onReachBottom() {
		/* 到底部加载 */
		this.fetchPageNum += 1
		this.getData()
		this.scroll.scrollTop -= 3300
		
	},
	onPullDownRefresh() {
		// 下拉刷新
		this.refreshing = true;
		this.getData();
		
	},
	onPageScroll: function(Object) {
	  this.scroll = Object;//实时获取到滚动的值 
	},
	
	methods: {
		checkstat() {
			var logininfo = util.checklogin();
			this.openid = logininfo.openid;
			this.userinfo = logininfo.userinfo;
			this.token = logininfo.token;
		},
		
		showreply(id, index) {
			// 进入回复页面
			uni.setStorage({
				key: 'detail-' + id,
				data: parSetData(this.list[index])
			});
			uni.navigateTo({
				url: '../detail/detail?id=' + id
			});
		},
		getData() {
			this.loading = true;
			uni.request({
				url: util.apiurl + 'api/v1/news_list',
				method: 'get',
				dataType: 'json',
				data: {
					page: this.refreshing ? 1 : this.fetchPageNum,
					pagesize: 5,
				},
				success: ret => {
					if (ret.statusCode !== 200) {
						uni.showToast({
							title: '请求数据失败请重试' + ret.statusCode,
							duration: 2000,
							icon: 'none'
						});
					} else {
						// console.log(ret.data.Result)
						// return
						if (ret.data.Result.length==0) {
							uni.showToast({
								title: '没有了',
								icon: 'none'
							});
							return false;
						}
						if (this.refreshing && ret.data.Result[0].newsid === this.list[0].id) {
							uni.showToast({
								title: '已经最新',
								icon: 'none'
							});
							this.refreshing = false;
							uni.stopPullDownRefresh();
							return;
						}
						if (this.refreshing) {
							this.refreshing = false;
							uni.stopPullDownRefresh();
							this.list = ret.data.Result;
							this.fetchPageNum = 2;
						} else {
							this.list = this.list.concat(ret.data.Result)
						}
					}
				},
				fail: res => {
					uni.showToast({
						title: '请求数据失败请重试',
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
		setlike(value) {
			uni.setStorageSync('detail', value)
			uni.navigateTo({
				url: '../detail/detail'
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
.iconbottom {
	align-items: center;
	text-align: center;
	justify-content: center;
	margin: 20upx auto 10upx;
}
.iconbottom .iconfont {
	display: inline-block;
	margin-left: 0upx;
	background-color: #00BFFF;
	font-size: 30upx;
}

.active {
	margin-top: 190upx;
	float: right;
	color: #666;
}
.card {
	padding: 30upx;
}
.card-img {
	width: 50%;
	height: 50%;
}
</style>
