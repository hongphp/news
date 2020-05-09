<template>
	<view class="index">
		<view class="card">
			<view class="car-title-view" v-if="data.c" v-html="data.c">
				
			</view>
	
			<view>
				<view
					@click="setlike('like', 0, data.newsid)"
					
					class="iconfont icon-appreciate_light zan"
				>
				<img src="../../static/点赞.png">
					<text>顶</text>
				</view>
				<view
					@click="setlike('hate', 0, data.newsid)"
					
					class="icon-oppose_light cai"
				>
					<img src="../../static/踩.png">
					<text>踩</text>
				</view>
			</view>
		</view>
		<view class="section">
			<form @submit="send">
				<textarea @focus="checklogin" v-model="text" placeholder="期待你的神评论" />
				<button form-type="submit" size="mini">发表评论</button>
			</form>
		</view>

		<view class="uni-list" v-if="replylist.length">
			<view class="uni-list-cell" v-for="(value, key) in replylist" :key="key">
				<view class="uni-media-list">
					<view class="uni-media-list-body">
						<view class="uni-media-list-text-bottom">
							<image
								class="uni-media-list-logo"
								:src="value.avatar ? value.avatar : '../../static/logo.png'"
							></image>
							<text>{{ value.nick }}</text>
							<text>评论于: {{ value.pub_time }}</text>
						</view>
						<view class="uni-media-list-text-top">{{ value.text }}</view>
					</view>
				</view>
			</view>
		</view>

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

<style>
@import '../../common/uni.css';
.section {
	display: block;
	margin-top: 120upx;
}
.section form {
	display: block;
	padding: 20upx;
}
.section textarea {
	background: #ffffff;
	display: block;
	margin: auto;
	font-size: 25upx;
	width: 88%;
	height: 100upx;
	padding: 10upx;
}
.section button {
	display: block;
	width: 200upx;
	margin: 10upx 0 10upx auto;
}

.uni-media-list-logo {
	width: 60upx;
	height: 60upx;
}

.uni-media-list-body {
	height: auto;
	justify-content: space-around;
}

.uni-media-list-text-top {
	font-size: 15upx;
	overflow: hidden;
	margin-top: 15upx;
	word-break: break-all;
}

.uni-media-list-text-bottom {
	display: flex;
	flex-direction: row;
	justify-content: flex-start;
	align-items: center;
}
.uni-media-list-text-bottom text {
	margin: auto 15upx;
}
.car-title-view {
	padding: 60upx
}
.uni-list {
	margin-bottom: 50upx;
}
.zan img, .cai img{
	width: 70upx;
}
.zan {
	float: left;
	margin-left: 220upx;
}
.cai {
	float: left;
	margin-left: 100upx;	
}
</style>

<script>
/* *
 *评论页面
 * */
import util,{ parGetData, parSetData, checklogin } from '../../common/util';
export default {
	data() {
		return {
			data: {},
			detailDec: '',
			text: '',
			id: '',
			showlogin: false,
			replylist: [],
			fetchPageNum: 1,
			openid: '',
			token: '',
			userinfo: '',
			loading: true
		};
	},
	onShow() {
		this.checkstat();
		this.fetchPageNum = 1;
		this.getData(this.id);
	},
	onLoad(e) {
		// e.data为获取的字符串
		// data 为传递来的数据
		this.id = e.newsid
		//比起使用vuex，更适合用uni.getStorage
		this.data = uni.getStorageSync('detail')
		console.log(this.data)
		this.data.c = this.data.c.replace(/<img/g, '<img width="100%"')
		this.loading = false
	
		
	},
	onShareAppMessage() {
		return {
			title: this.detailDec ? this.detailDec : '最新最好笑的趣事趣图，来看下吧',
			path: '/pages/detail/detail?id=' + this.id
		};
	},
	methods: {
		checkstat() {
			var logininfo = util.checklogin()
			this.openid = logininfo.openid;
			this.userinfo = logininfo.userinfo;
			this.token = logininfo.token;
		},
		checklogin() {
			if (!this.token) {
		
				uni.showModal({
					title: '请登录',
					content: '请登录后继续操作',
					confirmText: '去登录',
					success: function(res) {
						if (res.confirm) {
							uni.navigateTo({
							
								url: '/pages/login/login'
							})						
						}
					}
				});
				
				this.showlogin = true;
				return false;
			}
			
		},
		send(e) {		
			if (!this.text) {
				uni.showToast({
					title: '请输入评论内容',
					icon: 'none'
				});
				return false;
			}
			const self = this;
			self.replylist.unshift({
				text: this.text,
				avatar: this.userinfo.avatar,
				nick: this.userinfo.ddusername,
				pub_time: '刚刚'
			});
			uni.request({
				url: util.apiurl + 'api/v1/post_comment',
				method: 'POST',
				data: {
					text: self.text,
					id: self.id,
					token: self.token,
					shebei: util.shebei
				},
				success: ret => {
					
					if (ret.statusCode == 200 && ret.data.code == 1) {
						self.text = '';
						uni.showToast({
							title: '审核后显示'
						});
						setTimeout(function() {
							
						}, 1000);
					} else {
						uni.showToast({
							title: ret.data.msg,
							icon: 'none'
						});
						setTimeout(function() {
							
						}, 1000);
					}
				},
				fail: function() {
					uni.hideLoading();
				},
				complete: function() {
					this.loading = false;
				},
				header: {
					Authorization: util.httpauth
				}
			});
		},
		setlike(type, index, tid) {
			alert(1024)
		},
		onReachBottom() {
			/* 到底部加载 */
			this.getData(this.id);
		},
		getData(id) {
			const self = this;
			self.loading = true;
			// 获取评论
			uni.request({
				url: util.apiurl + 'api/v1/get_comment',
				method: 'get',
				dataType: 'json',
				data: {
					shebei: util.shebei
				},
				success: ret => {
					self.loading = false;
					if (ret.statusCode !== 200) {
						uni.showToast({
							title: '请求数据失败请重试' + ret.statusCode,
							duration: 2000,
							icon: 'none'
						});
					} else {
						if (self.fetchPageNum < 2) {
							// 这是刚进入时第一页
						} else if (ret.data.length < 1) {
							// 大于第一页且无评论
							uni.showToast({
								title: '没有更多评论了',
								icon: 'none'
							});
							return false;
						}
						// 第一页或者大于第一页且有评论时到这里
						self.replylist = ret.data;					
						self.fetchPageNum += 1;
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
					self.loading = false;
				},
				header: {
					Authorization: util.httpauth
				}
			});
		},
		
		/* 登录方法 */
		login(res) {
			util.logincomm(res, this);
		},
		
	}
};
</script>
