<template>
	<view>
		<!-- 		<view class="floatbox">
			<view class="floatbutton"></view>
			<view class="floatbutton"></view>
		</view> -->
		<view class="sendbutton" @click="openaddtieba()">

		</view>
		<adv :category_id="3"></adv>
		<view class="ltlistbox">
			<view class="ltitem ubb bc-border" @click="opentiebadetail(item.tieba_content_id)" v-for="item,index in luntanlist">
				<view class="left-box">
					<view class="headimage">
						<img :src="item.pics" style="width: 100%;height: 100%;">
					</view>
				</view>
				<view class="right-box">
					<view class="usernick">{{item.user_nick}}</view>
					<view class="sendtime">{{item.create_date.substr(5,5)}}</view>
					<view class="itemtitle" v-html="item.title"></view>
<!-- 					<view class="itemcontent" v-html="item.content">

					</view> -->
					<view class="itemcontentimage" v-if="item.pic.length!=0" v-for="itam in item.pic">
						<image :src="itam.upload_pic" mode="aspectFill" style="width: 100%;height: 100%;"></image>
					</view>
					<view class="footbox">
							<view class="replynum" style="color:#cdcdcd">
								<view class="commentbox"></view>
								<text class="replynum-text">
									{{item.reply_num}}
								</text>
							</view>
							<view class="like-box" @click.stop="changegoodtype(index,item.tieba_content_id)" >
								
							<view class="like" v-if="item.is_good==0"></view>
							<view class="like liked" v-if="item.is_good==1"></view>
							<text class="likenum" style="color:#cdcdcd">{{item.goods_num}}</text>
							</view>
					</view>
				</view>


			</view>
		</view>
		<uni-load-more :status="status" :content-text="contentText" />
	</view>
</template>

<script>
	import adv from '../../components/adv.vue'
	import slideshowHead from '../../components/slideshow-head.vue';
	import uniLoadMore from '../../components/uni-load-more/uni-load-more.vue'
	export default {
		components: {
			slideshowHead,
			uniLoadMore,
			adv
		},
		data() {
			return {
				//列表部分
				last_id: false,
				reload: false,
				status: 'more',
				luntanlist: [],
				contentText: {
					contentdown: '上拉加载更多',
					contentrefresh: '加载中',
					contentnomore: '没有更多内容了'
				},
				page: 1,
				recs: 10,
				slidearr: [{
					url: 'http://www.baidu.com',
					type: '',
					id: '',
					src: '../../static/radio/poster.png'
				}, {
					url: 'http://www.baidu.com',
					src: '../../static/radio/poster.png'
				}],
			}
		},
		onLoad() {
			this.getluntanlist()
		},
		onPullDownRefresh() {
			this.reload = true;
			this.last_id = false;
			this.luntanlist = [];
			this.page = 1;
			this.getluntanlist();
		},
		onReachBottom() {
			this.status = 'more';
			this.getluntanlist();
		},
		methods: {
			openaddtieba: function() {
				uni.navigateTo({
					url: '../dyzx-add/dyzx-add'
				})
			},
			opentiebadetail: function(a) {
				uni.setStorageSync("tiebaid", a);
				uni.navigateTo({
					url: '../dyzx-detail/dyzx-detail'
				})
				console.log(a)
			},
			changegoodtype: function(index, id) {
				console.log("点赞");
				// window.event ? window.event.cancelBubble = true : e.stopPropagation();
				uni.request({
					url: this.globalUrl + '/news/add_tieba_goods_num.php',
					data: {
						tieba_content_id: id,
						user_id: uni.getStorageSync("user_id"),
						is_new: 1
					},
					dataType: "json",
					method: "POST",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					success: (res) => {
						console.log(this.luntanlist[index])
						if (this.luntanlist[index].is_good == 1) { //取消点赞状态
							console.log("取消点赞状态");
							this.luntanlist[index].is_good = 0;
							this.luntanlist[index].goods_num--;
							console.log(this.luntanlist[index].is_good)
						} else { //修改为点赞状态
							console.log("点赞状态");
							this.luntanlist[index].is_good = 1;
							this.luntanlist[index].goods_num++;
							console.log(this.luntanlist[index].is_good)
						}
					}
				})
			},
			getluntanlist: function() {
				var that = this;
				if (that.last_id) {
					//说明已有数据，目前处于上拉加载
					that.status = 'loading';
					that.page++;
				}
				uni.request({
					url: that.globalUrl + '/news/get_tieba_content_by_category.php',
					data: {
						recs: that.recs,
						page: that.page,
						tieba_category_id: 4,
						user_id: uni.getStorageSync("user_id")
					},
					dataType: "json",
					method: "POST",
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					success: (res) => {
						var news = res.data;
						for (var i = 0; i < news.length; i++) {
							that.luntanlist.push(news[i]);
						}
						if (that.reload) {
							uni.stopPullDownRefresh();
						}
						that.last_id = true;
						that.reload = false;
						if (news.length < that.recs) {
							that.status = '';
						}

					}
				})
			}
		},
		onNavigationBarButtonTap(e) {
			console.log(e.index)
			if (e.index == 0) {
				uni.navigateTo({
					url: "../dyzx-mine/dyzx-mine"
				})
			}
		},
		onReachBottom() {
			// console.log("xxxxxxxxxxxxxxxxxxxxxxx");
			this.page++;
			this.getluntanlist()
		}

	}
</script>

<style>
	.sendbutton {
		position: fixed;
		bottom: 300rpx;
		right: 10rpx;
		width: 80rpx;
		height: 80rpx;
		background: url(../../static/dyzx/add.png) no-repeat center;
		background-size: 100% 100%;
		border-radius: 40rpx;
		z-index: 20;
	}


	.headadimage {
		position: relative;
		width: 100%;
		background-color: #000000;
		margin-bottom: 30rpx;
	}

	.floatbox {
		position: fixed;
		bottom: 0;
		right: 0;
		width: 250rpx;
		height: 100rpx;
		z-index: 10;
		display: flex;
		align-items: center;
		justify-content: space-around;
	}

	.ltitem {
		padding: 20rpx 20rpx 0 20rpx;
		position: relative;
		width: 100%;
		color: #444;
		/* border-bottom: 15rpx solid #f8f8f8; */
		display: flex;
	}

	.footbox {
		width: 100%;
		margin-top: 20rpx;
		background-color: #fff;
		display: flex;
		justify-content: flex-end;
		/* align-items: center; */
	}
	.like-box{
		height: 60rpx;
		margin-left: 20rpx;
	}
	.likenum .replynum-text {
		font-size: 28rpx;
		color: gray;
		margin-left: 10rpx;
	}

	.commentbox {
		display: inline-block;
		vertical-align:middle;
		background: url("../../static/dyzx/comment.png") no-repeat left;
		background-size: auto 100%;
		width: 28rpx;
		height: 28rpx;
		margin-right: 10rpx;
	}
	.like {
		display: inline-block;
		vertical-align:middle;
		background: url("../../static/dyzx/xin.png") no-repeat left;
		background-size: auto 80%;
		width: 28rpx;
		height: 28rpx;
		margin-right: 10rpx;
	}

	.liked {
		background: url("../../static/dyzx/xin1.png") no-repeat left;
		vertical-align:middle;
		background-size: auto 80%;
	}



	.sendtime {
		font-size: 20rpx;
		color: #b3b3b3;
	}

	.itemcontentimage {
		position: relative;
		width: calc(100% - 48rpx);
		height: 300rpx;
		background-color: #fff;
		margin-top: 20rpx;
		border-radius: 10rpx;
		overflow: hidden;
	}

/* 	.itemcontent {
		width: 100%;
		font-size: 24rpx;
		margin-top: 10rpx;
		color: gray;
		overflow: hidden;
		text-align: justify;
		text-overflow: ellipsis;
		display: -webkit-box;
		-webkit-line-clamp: 2;
		-webkit-box-orient: vertical;
	} */
	
	
	.headimage {
		position: relative;
		width: 70rpx;
		height: 70rpx;
		overflow: hidden;
		border: none;
		box-shadow: 0rpx 0rpx 20rpx 4rpx rgba(0, 0, 0, 0.1);
		background-color: #000000;
		border-radius: 80rpx;
	}

	.itemtitle {
		width: 100%;
		margin-top: 10rpx;
		font-size: 30rpx;
	}
	.right-box{
		width: 100%;
		padding: 0 20rpx 0 20rpx;
	}
</style>
