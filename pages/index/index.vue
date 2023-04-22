<template>
	<view class="indexContainer">
		<image src="@/static/index/background.jpg" class="bgImg" mode="aspectFill" />
		<view class="topBox">
			<view class="title">
				<view class="name" :class="{ 'select': item.selected }" v-for="(item, index) in imageList" :key="index" @click="itemClick(item)">
					{{ item.title }}
				</view>
			</view>
			<scroll-view scroll-x :show-scrollbar="false">
				<view class="imageCont">
					<image @click="imageClick(item, index)" v-for="(item, index) in getImageList()" :src="item.src" :key="index" />
				</view>
			</scroll-view>
		</view>
		<view class="mainBox">
			<view class="avatarBox " id="avatar-container">
				<image class="img" :src="avatarImage" v-if="avatarImage" />
				<image class="empty" src="/static/index/avatar.svg" v-else />
				<image class="avatarBg" :src="currentFrame" v-if="currentFrame" />
			</view>
			<view class="btnBox">
				<!-- #ifdef H5 -->
				<view class="iconBtn">
					<view class="icon-zuo iconfont" v-if="showSwitch(-1)" @click="switchAvatar(-1)"></view>
					<view class="icon-you iconfont" v-if="showSwitch(1)" @click="switchAvatar(1)"></view>
				</view>
				<!-- #endif -->
				<button class="actionBtn" @click="uploadFace()">上传头像</button>
				<!-- #ifdef MP-WEIXIN -->
				<button class="actionBtn" open-type="chooseAvatar" @chooseavatar="onChooseAvatar">获取头像</button>
				<!-- #endif -->
				<button class="actionBtn save" @click="saveFace()">保存头像</button>
			</view>
		</view>
		<view class="hideCanvasView">
			<canvas class="hideCanvas" :id="canvasId" :canvas-id="canvasId" :style="{ width: (poster.width || 10) + 'px', height: (poster.height || 10) + 'px' }"></canvas>
		</view>
	</view>
</template>
<script>
	import { getSharePoster } from '@/util/createPoster/createPoster.js'
	export default {
		data() {
			return {
				poster: {},
				posterImage: '', //生成的图片
				canvasId: 'defaultCanvasId', //画布id
				userInfo: '',
				code: '',
				readLocal: true, //是否读取缓存信息
				avatarImage: '', //头像
				currentFrame: '/static/images/gradual/1.png',
				currentIndex: 0,
				imageList: [{
					title: '渐变国旗',
					selected: true,
					imgs: [{ src: '/static/images/gradual/1.png' }, { src: '/static/images/gradual/2.png' }, { src: '/static/images/gradual/3.png' }, { src: '/static/images/gradual/4.png' }, { src: '/static/images/gradual/5.png' }, { src: '/static/images/gradual/6.png' }]
				}, {
					title: '多样背景',
					selected: false,
					imgs: [{ src: '/static/images/other/1.png' }, { src: '/static/images/other/2.png' }, { src: '/static/images/other/3.png' }, { src: '/static/images/other/4.png' }, { src: '/static/images/other/5.png' }, { src: '/static/images/other/6.png' }, { src: '/static/images/other/7.png' },
						{ src: '/static/images/other/8.png' }, { src: '/static/images/other/9.png' }, { src: '/static/images/other/10.png' }
					]
				}, {
					title: '质朴国旗',

					selected: false,
					imgs: [{ src: '/static/images/simple/1.png' }, { src: '/static/images/simple/2.png' }, { src: '/static/images/simple/3.png' }, { src: '/static/images/simple/4.png' }, { src: '/static/images/simple/5.png' }, { src: '/static/images/simple/6.png' }, { src: '/static/images/simple/7.png' }],
				}, {
					title: '国庆新款',
					selected: false,
					imgs: [{ src: '/static/images/new/1.png' }, { src: '/static/images/new/2.png' }, { src: '/static/images/new/3.png' }]
				}]
			}
		},
		onLoad() {
			// #ifdef MP-WEIXIN
			let that = this;
			uni.login({
				provider: 'weixin',
				success: function(loginRes) {
					that.code = loginRes.code;
				},
				fail: function(result) {}
			});
			// #endif
			if (this.readLocal) {
				this.userInfo = uni.getStorageSync('user_info')
				this.avatarImage = this.userInfo.avatarUrl
			}
		},
		//小程序分享
		onShareAppMessage: function() {
			return {
				title: '我刚刚换上了国庆头像，你也来领取一个吧',
				desc: '领取你的国庆头像，为祖国加油',
				imageUrl: '/static/index/share.jpg',
				path: '/pages/index/index',
				success: function(e) {
					console.log(e)
				}
			}
		},
		onShareTimeline: function() {
			return {
				title: '我刚刚换上了国庆头像，你也来领取一个吧',
				desc: '领取你的国庆头像，为祖国加油',
				imageUrl: '/static/index/share.jpg',
				path: '/pages/index/index',
				success: function(e) {
					console.log(e)
				}
			}
		},
		methods: {
			switchAvatar(num) {
				if ((num > 0 && this.currentIndex < this.getImageList().length - 1) || (num < 0 && this.currentIndex > 0)) {
					this.currentIndex += num
					this.currentFrame = this.getImageList()[this.currentIndex].src
				} else {
					let currentType = this.imageList.findIndex(data => data.selected)
					currentType += num
					if (currentType >= 0 && currentType < this.imageList.length) {
						this.itemClick(this.imageList[currentType])
						this.currentIndex = num < 0 ? this.getImageList().length - 1 : 0
						this.currentFrame = this.getImageList()[0].src
					}
				}
			},
			showSwitch(val) {
				let currentType = this.imageList.findIndex(data => data.selected)
				let res = (val < 0 && currentType <= 0 && this.currentIndex <= 0) || (val > 0 && currentType >= this.imageList.length - 1 && this.currentIndex >= this.getImageList().length - 1);
				return !res
			},
			imageClick(item, index) {
				this.currentIndex = index
				this.currentFrame = item.src
			},
			//取选中项的图片
			getImageList() {
				let item = this.imageList.filter(data => {
					return data.selected;
				})
				return item[0].imgs;
			},
			itemClick(item) {
				this.currentIndex = 0;
				this.imageList.forEach(data => {
					data.selected = false
				});
				item.selected = true
			},
			uploadFace() {
				let that = this
				const crop = {
					quality: 100,
					width: 600,
					height: 600,
					resize: true
				};
				uni.chooseImage({
					count: 1,
					crop,
					sourceType: ['album'],
					success: async (res) => {
						let size = res.tempFiles[0].size
						if (res.tempFiles[0].size / 1024 / 1024 > 2) {
							uni.showToast({
								title: '图片大小不能大于2M',
								icon: 'none',
								mask: true
							})
						} else {
							let filePath = res.tempFilePaths[0]
							// #ifndef APP-PLUS
							filePath = await new Promise((callback) => {
								uni.navigateTo({
									url: '/pages/index/cropImage?path=' + filePath + `&options=${JSON.stringify(crop)}`,
									animationType: "fade-in",
									events: {
										success: url => {
											callback(url)
										}
									}
								});
							})
							that.avatarImage = filePath
							// #endif
						}
					}
				})
			},
			//原来的uni.getUserProfile已经不可用,此方法获取头像非常模糊,官方没有目前还未解决,建议自己上传图片
			onChooseAvatar(e) {
				const { avatarUrl } = e.detail
				const userInfo = {
					avatarUrl
				}
				this.avatarImage = avatarUrl
				uni.setStorageSync('user_info', userInfo)
			},
			async saveFace() {
				if (!this.avatarImage) {
					uni.showToast({
						title: '请先上传头像',
						icon: 'error',
						mask: true
					})
					return;
				}
				try {
					uni.showLoading({
						title: '图片生成中..',
						mask: true
					})
					const d = await getSharePoster({
						that: this, //若在组件中使用必传
						type: 'testShareType',
						formData: {
							//访问接口获取背景图携带自定义数据
						},
						backgroundImage: this.avatarImage,
						posterCanvasId: this.canvasId, //canvasId
						delayTimeScale: 20, //延时系数
						drawArray: ({
							bgObj,
							type,
							bgScale
						}) => {
							//可直接return数组，也可以return一个promise对象, 但最终resolve一个数组, 这样就可以方便实现后台可控绘制海报
							return new Promise((rs, rj) => {
								rs([{
									type: 'image',
									url: this.currentFrame,
									dx: 0,
									dy: 0,
									infoCallBack(imageInfo) {
										return {
											dWidth: bgObj.width, // 因为设置了圆形图片 所以要乘以2
											dHeight: bgObj.height
										}
									}
								}])
							})
						},
						setCanvasWH: ({
							bgObj,
							type,
							bgScale
						}) => {
							// 为动态设置画布宽高的方法，
							this.poster = bgObj;
						}
					})
					this.posterImage = d.poster.tempFilePath
					// #ifdef H5
					//h5生成的图片需要长按保存
					uni.hideLoading()
					uni.previewImage({
						urls: [this.posterImage]
					})
					// #endif
					// #ifdef MP-WEIXIN
					this.savefile()
					// #endif
				} catch (e) {
					uni.hideLoading()
				}
			},
			//获取相册授权
			savefile() {
				let that = this;
				uni.getSetting({
					success(res) {
						if (!res.authSetting['scope.writePhotosAlbum']) {
							uni.authorize({
								scope: 'scope.writePhotosAlbum',
								success() {
									//这里是用户同意授权后的回调
									that.saveImgToLocal()
								},
								fail(e) {
									uni.hideLoading()
									wx.showModal({
										content: '检测到您没打开下载图片功能权限，是否去设置打开？',
										confirmText: '确认',
										cancelText: '取消',
										success: function(res) {
											//点击“确认”时打开设置页面
											if (res.confirm) {
												wx.openSetting()
											}
										}
									})
								}
							})
						} else {
							//用户已经授权过了
							that.saveImgToLocal()
						}
					}
				})
			},
			saveImgToLocal(e) {
				let that = this
				uni.saveImageToPhotosAlbum({
					filePath: that.posterImage,
					success: function() {
						uni.hideLoading()
						uni.showToast({
							title: '保存成功',
							icon: 'success'
						})
					},
					fail: function(err) {
						uni.hideLoading()
						uni.showToast({
							title: err.errMsg.includes("cancel") ? "用户取消" : "保存失败",
							icon: 'error'
						})
					}
				})
			}
		}
	}
</script>
<style lang="scss">
	.indexContainer {
		height: 100vh;
		padding-top: 200upx;
		box-sizing: border-box;
		overflow: hidden;
		.bgImg {
			position: absolute;
			z-index: -1;
			top: 0;
			left: 0;
			right: 0;
			bottom: 0;
			height: 100vh;
			width: 100%;
			background-color: #ffd48f;
		}
		.topBox {
			background-color: #ffffff;
			margin: 0 30upx;
			border-radius: 40upx;
			padding: 40upx 45upx 40upx;
			position: relative;
			.title {
				display: flex;
				align-items: center;
				justify-content: space-evenly;
				padding: 0 0 40upx;
				.name {
					flex: 1;
					font-size: 30upx;
					text-align: center;
				}
				.select {
					font-weight: bold;
					color: #ff4500;
				}
			}
			.imageCont {
				display: flex;
				align-items: center;
				image {
					width: 120upx;
					height: 120upx;
					flex-shrink: 0;
					margin: 0 15upx;
					&:last-child {
						padding-right: 15upx;
					}
				}
			}
		}
		.mainBox {
			display: flex;
			justify-content: center;
			justify-content: space-between;
			margin: 60upx 30upx 0;
			.avatarBox {
				position: relative;
				height: 380upx;
				width: 380upx;
				margin-right: 40upx;
				display: flex;
				align-items: center;
				justify-content: center;
				flex-direction: column;
				background-color: #fff;
				border-radius: 40upx;
				overflow: hidden;
				.img {
					height: 100%;
					width: 100%;
				}
				.empty {
					height: 100px;
					width: 100px;
				}
				.avatarBg {
					position: absolute;
					left: 0;
					top: 0;
					z-index: 10;
					height: 100%;
					width: 100%;
				}
			}
			.btnBox {
				display: flex;
				flex-direction: column;
				justify-content: space-between;
				padding: 10upx 0;
				.iconBtn {
					position: relative;
					height: 80upx;
					.iconfont {
						color: #f7f8fa;
						font-size: 80upx;
						font-weight: bold;
					}
					.icon-zuo {
						position: absolute;
						left: 0;
					}
					.icon-you {
						position: absolute;
						right: 0;
					}
				}
				.actionBtn {
					background: #fff;
					border: none;
					border-radius: 48upx;
					box-shadow: 0 12upx 16upx -8upx rgba(0, 0, 0, 0.1);
					color: #4d4d4d;
					font-weight: bolder;
					height: 90upx;
					line-height: 90upx;
					font-size: 30upx;
					padding: 0 60upx;
					&::after {
						border: none;
					}
					&.save {
						background: linear-gradient(97.71deg, #ffa462, #ff4d42 88.36%);
						border-radius: 48upx;
						box-shadow: 0 12upx 16upx -8upx rgba(255, 88, 35, 0.6);
						color: #fff;
					}
				}
			}
		}
	}
	.hideCanvasView {
		position: relative;
		.hideCanvas {
			position: fixed;
			top: -9999upx;
			left: -9999upx;
			z-index: -9999;
		}
	}
</style>