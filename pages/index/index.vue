<template>
	<view class="content">
		<image src="../../static/background.jpg" class="all-back"></image>
		<view class="top-content">
			<view class="top-title">
				<view class="title-unit" :class="{ 'title-select': item.selected }" v-for="(item, index) in imageList" :key="item.key" @click="itemClick(item)">
					{{ item.title }}
				</view>
			</view>
			<scroll-view scroll-x :show-scrollbar="false" class="scroll-view">
				<view class="image-div">
					<image
						:class="{ 'image-margin': index !== 0 }"
						@click="imageClick(item, index)"
						v-for="(item, index) in getImageList()"
						:src="item.src"
						:key="index"
					></image>
				</view>
			</scroll-view>
		</view>

		<view class="image-card">
			<view class="photo-main-view">
				<!--  -->
				<view class="avatar-div " id="avatar-container">
					<image class="img" id="avatar-img" :src="avatarImage"></image>

					<view class="empty-view " v-if="!avatarImage"><image class="empty"
					 src="../../static/images/avatar_empty.svg"></image></view>

					<image class="avatar-default " :src="currentFrame" v-if="currentFrame"></image>
				</view>

				<view class="ctlbtn">
					<view class="icon-div btn-margin">
						<view class="icon-zuo iconfont" v-if="showSwitch(-1)" @click="switchAvatar(-1)"></view>
						<view class="icon-you iconfont" v-if="showSwitch(1)" @click="switchAvatar(1)"></view>
					</view>
					<view v-if="userInfo" class="action-btn btn-margin" @click="getUserProfile()">获取头像</view>
					<button class="action-btn btn-margin" v-else open-type="getUserInfo" @click="getUserProfile()">获取头像</button>
					<button class="action-btn btn-primary" @click="shareFc()">保存头像</button>
				</view>
			</view>
		</view>

		<view class="hideCanvasView">
			<canvas
				class="hideCanvas"
				id="default_PosterCanvasId"
				canvas-id="default_PosterCanvasId"
				:style="{ width: (poster.width || 10) + 'px', height: (poster.height || 10) + 'px' }"
			></canvas>
		</view>
	</view>
</template>

<script>
import _app from '@/util/QS-SharePoster/app.js';
import { getSharePoster } from '@/util/QS-SharePoster/QS-SharePoster.js';
const IMAGE_LIST = [
	{
		key: '1',
		title: '国庆新款',
		selected: true,
		imageList: [
			{
				key: '1.1',
				src: '../../static/images/new/NA.png'
			},
			{
				key: '1.2',
				src: '../../static/images/new/NB.png'
			},
			{
				key: '1.3',
				src: '../../static/images/new/NC.png'
			},
			{
				key: '1.4',
				src: '../../static/images/new/ND.png'
			},
			{
				key: '1.9',
				src: '../../static/images/new/NE.png'
			},
			{
				key: '1.6',
				src: '../../static/images/new/NF.png'
			},
			{
				key: '1.7',
				src: '../../static/images/new/NG.png'
			},
			{
				key: '1.8',
				src: '../../static/images/new/NH.png'
			},
			{
				key: '1.9',
				src: '../../static/images/new/NI.png'
			},
			{
				key: '1.10',
				src: '../../static/images/new/NJ.png'
			}
		]
	},
	{
		key: '2',
		title: '渐变国旗',
		selected: false,
		imageList: [
			{
				key: '1.1',
				src: '../../static/images/gradual/1.png'
			},
			{
				key: '1.2',
				src: '../../static/images/gradual/2.png'
			},
			{
				key: '1.3',
				src: '../../static/images/gradual/3.png'
			},
			{
				key: '1.4',
				src: '../../static/images/gradual/4.png'
			},
			{
				key: '1.5',
				src: '../../static/images/gradual/5.png'
			},
			{
				key: '1.6',
				src: '../../static/images/gradual/6.png'
			}
		]
	},
	{
		key: '3',
		title: '质朴国旗',
		selected: false,
		imageList: [
			{
				key: '1.1',
				src: '../../static/images/simple/2.png'
			},
			{
				key: '1.2',
				src: '../../static/images/simple/3.png'
			},
			{
				key: '1.3',
				src: '../../static/images/simple/4.png'
			},
			{
				key: '1.4',
				src: '../../static/images/simple/5.png'
			},
			{
				key: '1.5',
				src: '../../static/images/simple/6.png'
			},
			{
				key: '1.6',
				src: '../../static/images/simple/7.png'
			},
			{
				key: '1.7',
				src: '../../static/images/simple/8.png'
			}
		]
	},
	{
		key: '4',
		title: '其他',
		selected: false,
		imageList: [
			{
				key: '1.1',
				src: '../../static/images/other/1.png'
			},
			{
				key: '1.2',
				src: '../../static/images/other/2.png'
			}
		]
	}
];
export default {
	data() {
		return {
			poster: {},
			posterImage: '',
			canvasId: 'default_PosterCanvasId',
			userInfo: '',
			code: '',
			avatarImage: '',
			currentFrame: '../../static/images/new/NA.png',
			currentIndex: 0,
			imageList: IMAGE_LIST
		};
	},
	onLoad() {
		// #ifdef MP-WEIXIN
		let _this = this;
		uni.login({
			provider: 'weixin',
			success: function(loginRes) {
				_this.code = loginRes.code;
			},
			fail: function(result) {}
		});
		this.init();
		// #endif
	},
	onShareAppMessage: function() {
		return {
			title: '我刚刚换上了国庆头像，你也来领取一个吧',
			desc: '领取你的国庆头像，为祖国加油',
			imageUrl: '/static/share.jpg',
			path: '/pages/index/index',
			success: function(e) {
				console.log(e);
			}
		};
	},
	onShareTimeline: function() {
		return {
			title: '我刚刚换上了国庆头像，你也来领取一个吧',
			desc: '领取你的国庆头像，为祖国加油',
			imageUrl: '/static/share.jpg',
			path: '/pages/index/index',
			success: function(e) {
				console.log(e);
			}
		};
	},
	methods: {
		switchAvatar(num) {
			if ((num > 0 && this.currentIndex < this.getImageList().length - 1) || (num < 0 && this.currentIndex > 0)) {
				this.currentIndex += num;
				this.currentFrame = this.getImageList()[this.currentIndex].src;
			} else {
				let currentType = this.imageList.findIndex(data => data.selected);
				currentType += num;
				if (currentType >= 0 && currentType < this.imageList.length) {
					this.itemClick(this.imageList[currentType]);
					this.currentIndex = num < 0 ? this.getImageList().length - 1 : 0;
					this.currentFrame = this.getImageList()[0].src;
				}
			}
		},
		showSwitch(val) {
			let currentType = this.imageList.findIndex(data => data.selected);
			let res =
				(val < 0 && currentType <= 0 && this.currentIndex <= 0) ||
				(val > 0 && currentType >= this.imageList.length - 1 && this.currentIndex >= this.getImageList().length - 1);
			return !res;
		},
		async shareFc() {
			if (!this.avatarImage) {
				uni.showToast({
					title: '请先获取头像',
					icon: 'none'
				});
				return;
			}
			try {
				uni.showLoading({
					title: '加载中'
				});
				_app.log('准备生成:' + new Date());
				const d = await getSharePoster({
					_this: this, //若在组件中使用 必传
					type: 'testShareType',
					formData: {
						//访问接口获取背景图携带自定义数据
					},
					backgroundImage: this.avatarImage,
					posterCanvasId: this.canvasId, //canvasId
					delayTimeScale: 20, //延时系数
					drawArray: ({ bgObj, type, bgScale }) => {
						//可直接return数组，也可以return一个promise对象, 但最终resolve一个数组, 这样就可以方便实现后台可控绘制海报
						return new Promise((rs, rj) => {
							rs([
								{
									type: 'image',
									url: this.currentFrame,
									dx: 0,
									dy: 0,
									infoCallBack(imageInfo) {
										return {
											dWidth: bgObj.width, // 因为设置了圆形图片 所以要乘以2
											dHeight: bgObj.height
										};
									}
								}
							]);
						});
					},
					setCanvasWH: ({ bgObj, type, bgScale }) => {
						// 为动态设置画布宽高的方法，
						this.poster = bgObj;
					}
				});
				_app.log('海报生成成功, 时间:' + new Date() + '， 临时路径: ' + d.poster.tempFilePath);
				this.posterImage = d.poster.tempFilePath;
				this.savefile();
			} catch (e) {
				uni.hideLoading();
				_app.hideLoading();
				_app.showToast(JSON.stringify(e));
			}
		},
		saveImage() {
			// #ifndef H5
			uni.saveImageToPhotosAlbum({
				filePath: this.poster.finalPath,
				success(res) {
					uni.showToast({
						title: '保存成功'
					});
				}
			});
			// #endif
		},
		savefile() {
			//获取相册授权
			let _self = this;
			uni.getSetting({
				success(res) {
					if (!res.authSetting['scope.writePhotosAlbum']) {
						uni.authorize({
							scope: 'scope.writePhotosAlbum',
							success() {
								//这里是用户同意授权后的回调
								_self.saveImgToLocal();
							},
							fail(e) {
								uni.hideLoading();
								wx.showModal({
									content: '检测到您没打开下载图片功能权限，是否去设置打开？',
									confirmText: '确认',
									cancelText: '取消',
									success: function(res) {
										//点击“确认”时打开设置页面
										if (res.confirm) {
											wx.openSetting();
										}
									}
								});
							}
						});
					} else {
						//用户已经授权过了
						_self.saveImgToLocal();
					}
				}
			});
		},
		saveImgToLocal(e) {
			let _self = this;
			// uni.downloadFile({
			// 	url: _self.posterImage, //图片地址
			// 	success: res => {
			// 		if (res.statusCode === 200) {
						uni.saveImageToPhotosAlbum({
							filePath: _self.posterImage,
							success: function() {
								uni.hideLoading();
								uni.showToast({
									title: '保存成功',
									icon: 'none'
								});
							},
							fail: function(AAAA) {
								uni.hideLoading();
								uni.showToast({
									title: '保存失败',
									icon: 'none'
								});
							}
						});
			// 		}
			// 	},
			// 	fail: res => {
			// 		uni.hideLoading();
			// 	}
			// });
		},
		getNetworkImage(url) {
			return new Promise((resolve, reject) => {
				uni.downloadFile({
					url,
					success: e => {
						const p = e.tempFilePath; //这个就是jpg地址
						this.pic0 = p;
						if (p.indexOf('json') > -1) {
							reject(p);
							return false;
						}
						resolve(p);
					},
					fail: r => {
						reject(r);
					}
				});
			});
		},
		imageClick(item, index) {
			this.currentIndex = index;
			this.currentFrame = item.src;
		},
		getImageList() {
			let item = this.imageList.filter(data => {
				return data.selected;
			});
			return item[0].imageList;
		},
		itemClick(item) {
			this.currentIndex = 0;
			this.imageList.forEach(data => {
				data.selected = false;
			});
			item.selected = true;
		},
		init() {
			this.userInfo = uni.getStorageSync('user_info');
		},
		getUserProfile(e) {
			// #ifdef MP-WEIXIN
			let _this = this;
			uni.getUserProfile({
				desc: '获取您的头像信息',
				success(result) {
					let data = {
						code: _this.code,
						signature: result.signature,
						encrypted_data: result.encryptedData,
						iv: result.iv,
						userInfo: result.userInfo
					};
					let info = data.userInfo.avatarUrl;
					_this.avatarImage = info.substring(0, info.lastIndexOf('/') + 1) + '0';
					uni.setStorageSync('user_info', data.userInfo);
					_this.init();
				},
				fail(fall) {}
			});
			// #endif
			// #ifndef MP-WEIXIN
			uni.showToast({
				title:'请在微信开发工具运行',
				icon:'none'
			})
			// #endif
		}
	}
};
</script>

<style lang="scss">
.content {
	background-size: 100% 100%;
	padding-top: 200rpx;
	
	.all-back {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		min-height: 100vh;
		width: 100%;
	}
	.top-content {
		width: 610rpx;
		background-color: #ffffff;
		margin: 30rpx;
		border-radius: 50rpx;
		padding: 0 40rpx 30rpx;
		position: relative;
		.top-title {
			display: flex;
			align-items: center;
			.title-unit {
				padding: 40rpx 20rpx;
				font-size: 30rpx;
			}
			.title-select {
				font-size: 30rpx;
				font-weight: bold;
				color: #ff4500;
			}
		}
		.image-div {
			display: flex;
			align-items: center;
			padding-left: 20rpx;
			padding-bottom: 20rpx;
			background-color: #ffffff;
			image {
				width: 120rpx;
				height: 120rpx;
				border: 1rpx solid #f8f8f8;
				box-shadow: 0px -5px 15px 0px rgba(224, 224, 224, 0.4);
				flex-shrink: 0;
			}
			.image-margin {
				margin: 0 20rpx;
			}
		}
	}
	.image-card {
		display: flex;
		justify-content: center;
		align-items: center;
		position: relative;
		.image-center {
			width: 300rpx;
			height: 300rpx;
			border-radius: 50rpx;
			margin: 0 70rpx;
		}
		.iconfont {
			color: #f7f8fa;
			font-size: 80rpx;
			font-weight: bold;
		}
	}
	.btn-div {
		padding: 50rpx;
		display: flex;
		justify-content: space-between;
		.btn-left {
			background-color: #f7f8fa;
			box-shadow: 0px 4px 54px 0px rgba(108, 108, 108, 0.14);
			padding: 0 70rpx;
			height: 100rpx;
			line-height: 100rpx;
			border-radius: 80rpx;
			color: #646566;
			font-weight: bold;
		}
		.btn-right {
			background-image: linear-gradient(90deg, #ff8c00, #ff4500);
			padding: 0 100rpx;
			height: 100rpx;
			line-height: 100rpx;
			border-radius: 80rpx;
			color: #ffffff;
			font-weight: bold;
		}
	}
}

.title {
	font-size: 36rpx;
	color: #8f8f94;
}

.avatar-div {
	height: 380rpx;
	margin-right: 40rpx;
	position: relative;
	width: 380rpx;
}

.avatar-div,
.empty-view {
	-webkit-box-orient: vertical;
	-webkit-box-direction: normal;
	-webkit-box-pack: center;
	-webkit-box-align: center;
	align-items: center;
	display: flex;
	flex-direction: column;
	justify-content: center;
	z-index: 1;
}

.empty {
	height: 100px;
	margin-bottom: 24px;
	width: 100px;
}

.img {
	background-color: #fff;
	border-radius: 48rpx;
	height: 360rpx;
	position: absolute;
	width: 360rpx;
	z-index: 0;
}

.avatar-default {
	border-radius: 48rpx;
	height: 100%;
	left: 0;
	position: absolute;
	top: 0;
	width: 100%;
	z-index: 10;
}

.container {
	background-color: #fbebe1;
	min-height: 100vh;
	overflow: hidden;
}
.photo-main-view {
	display: flex;
	justify-content: space-between;
	width: 690rpx;
	margin: 30rpx 30rpx 0;
}
.icon-div {
	position: relative;
	height: 80rpx;
	.icon-zuo {
		position: absolute;
		left: 0;
	}
	.icon-you {
		position: absolute;
		right: 0;
	}
}
.action-btn {
	background: #fff;
	border: 1rpx solid #efefef;
	border-radius: 48rpx;
	box-shadow: 0 12rpx 16rpx -8rpx rgba(0, 0, 0, 0.1);
	color: #4d4d4d;
	font-weight: bolder;
	height: 90rpx;
	line-height: 90rpx;
	font-size: 30rpx;
	padding: 0 60rpx;
}
.btn-margin {
	margin-bottom: 50rpx;
}
.btn-primary {
	background: linear-gradient(97.71deg, #ffa462, #ff4d42 88.36%);
	border: 1rpx solid #ff7852;
	border-radius: 48rpx;
	box-shadow: 0 12rpx 16rpx -8rpx rgba(255, 88, 35, 0.6);
	color: #fff;
}

.hideCanvasView {
	position: relative;
}

.hideCanvas {
	position: fixed;
	top: -99999upx;
	left: -99999upx;
	z-index: -99999;
}
</style>
