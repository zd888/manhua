<template>
	<view class="container">
		<!-- 小程序状态下登录 -->
		<!-- #ifdef MP-WEIXIN -->
		<view class="mp_wxBox">
			<view>
				<view class="headers">
					<image src="../../static/logo.png" style="border-radius: 50%;"></image>
				</view>
				<view class="content">
					<view>申请获取以下权限</view>
					<text>获得你的公开信息(昵称，头像、地区等)</text>
				</view>
				<button v-show="weixinPhone" style="background: #ffe121;color: #333;" class="bottom"
					open-type="getPhoneNumber" @getphonenumber="getPhoneNumber">
					授权手机号
				</button>
				<button v-show="!weixinPhone" style="background: #ffe121;color: #333;" class='bottom'
					bindtap="getUserProfile" @tap="wxGetUserInfo">
					授权登录
				</button>
			</view>

		</view>
		<!-- #endif -->

		<!-- #ifndef MP-WEIXIN -->
		<view style="text-align: center;">
			<image style="width: 120upx;height: 120upx;margin-top: 140upx;border-radius:20upx"
				src="../../static/logo.png"></image>

			<button class='confirm-btn' v-if="!isopen" @click="register">手机号登录</button>
			<button class='confirm-btn' v-if="isopen" @click="bingwx">微信一键登录</button>
		</view>
		<!-- #endif -->

		<view class="footer" @click.stop="isCheck">
			<view style="display: flex;align-items: center;justify-content: center;">
				<u-checkbox-group shape="circle" v-model="checked" placement="row">
					<u-checkbox activeColor="#ffe121" :name="true">
					</u-checkbox>
				</u-checkbox-group>
				<view>同意</view>
				<!-- 协议地址 -->
				<text @click.stop="goTo('/package/setting/mimi')">《隐私政策》</text>和<text
					@click.stop="goTo('/package/setting/xieyi')">《用户协议》</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				mobile: '',
				code: '',
				sessionkey: '',
				flag: '1',
				weixinPhone: false,
				sending: false,
				sendDataList: {},
				phone: '',
				sendTime: '获取验证码',
				count: 60,
				checked: false,
				isopen: false,
			};
		},
		onLoad() {
			// this.$u.get('/app/common/type/108').then(res => { //// 是否开启公众号自动登陆 108
			// 	if (res.code == 0 && res.data) {
			// 		if (res.data.value == '是') {
			// 			// #ifdef H5
			// 			let ua = navigator.userAgent.toLowerCase();
			// 			if (ua.indexOf('micromessenger') !== -1) {
			// 				this.isopen = true;
			// 			} else {
			// 				this.isopen = false;
			// 			}
			// 			// #endif
			// 		} else {
			// 			this.isopen = false;
			// 		}
			// 	}
			// })
			// // #ifdef H5
			// this.selbindwx();
			// // #endif
		},
		methods: {
			goTo(url) {
				uni.navigateTo({
					url
				});
			},
			isCheck() {
				this.checked = !this.checked
			},
			selbindwx() {
				let ua = navigator.userAgent.toLowerCase();
				if (ua.indexOf('micromessenger') !== -1) {
					let openid = uni.getStorageSync('openid');
					let userId = uni.getStorageSync('userId');
					let that = this;
					if (!openid) {
						if (window.location.href.indexOf('?code=') !== -1 || window.location.href.indexOf('&code=') !==
							-1) {
							let code;
							if (window.location.href.indexOf('?code=') !== -1) {
								code = window.location.href.split('?code=')[1].split('&')[0];
							} else {
								code = window.location.href.split('&code=')[1].split('&')[0];
							}
							this.$Request.get('/app/Login/getOpenId?code=' + code).then(ret => {
								uni.setStorageSync('openid', ret.data)

								this.$Request.get('/app/Login/openid/login?openId=' + ret.data).then(res => {
									if (res.code == 0) {
										this.$queue.setData("phone", res.user.phone);
										this.$queue.setData("userId", res.user.userId);
										this.$queue.setData("token", res.token);
										uni.setStorageSync('sex', res.user.sex)
										uni.setStorageSync('openId', res.user.openId)
										this.$queue.setData("userName", res.user.userName);
										this.$queue.setData("avatar", res.user.avatar);
										this.$queue.setData("invitationCode", res.user.invitationCode);
										this.$queue.setData("inviterCode", res.user.inviterCode);
										uni.switchTab({
											url: '/pages/index/index'
										})
									} else {
										uni.navigateTo({
											url: '/pages/login/bind'
										});
									}
								});
							});
						}
					}
				}
			},
			// 微信公众号登录
			bingwx() {
				if (this.checked) {

					let ua = navigator.userAgent.toLowerCase();
					if (ua.indexOf('micromessenger') !== -1) {
						let openid = uni.getStorageSync('openid');
						let userId = uni.getStorageSync('userId');
						let that = this;
						if (!openid) {
							if (window.location.href.indexOf('?code=') !== -1 || window.location.href.indexOf(
									'&code=') !==
								-1) {
								let code;
								if (window.location.href.indexOf('?code=') !== -1) {
									code = window.location.href.split('?code=')[1].split('&')[0];
								} else {
									code = window.location.href.split('&code=')[1].split('&')[0];
								}
								this.$Request.get('/app/Login/getOpenId?code=' + code).then(ret => {
									uni.setStorageSync('openid', ret.data)
									// uni.setStorageSync('openidheadimgurl', ret.data.headimgurl)
									// uni.setStorageSync('openidnickname', ret.data.nickname)

									this.$Request.get('/app/Login/openid/login?openId=' + ret.data).then(
										res => {
											if (res.code == 0) {
												this.$queue.setData("phone", res.user.phone);
												this.$queue.setData("userId", res.user.userId);
												this.$queue.setData("token", res.token);
												uni.setStorageSync('sex', res.user.sex)
												uni.setStorageSync('openId', res.user.openId)
												this.$queue.setData("userName", res.user.userName);
												this.$queue.setData("avatar", res.user.avatar);
												this.$queue.setData("invitationCode", res.user
													.invitationCode);
												this.$queue.setData("inviterCode", res.user
													.inviterCode);
												uni.switchTab({
													url: '/pages/index/index'
												})
											} else {
												uni.navigateTo({
													url: '/pages/login/bind'
												});
											}
										});
								});
							} else {
								window.location.href =
									'https://open.weixin.qq.com/connect/oauth2/authorize?appid=' +
									that.$queue.getWxAppid() +
									'&redirect_uri=' +
									window.location.href.split('#')[0] +
									'&response_type=code&scope=snsapi_userinfo#wechat_redirect';
							}
						} else {
							this.$Request.get('/app/Login/openid/login?openId=' + openid).then(res => {
								if (res.code == 0) {
									this.$queue.setData("phone", res.user.phone);
									this.$queue.setData("userId", res.user.userId);
									this.$queue.setData("token", res.token);
									this.$queue.setData("userName", res.user.userName);
									this.$queue.setData("avatar", res.user.avatar);
									this.$queue.setData("invitationCode", res.user.invitationCode);
									this.$queue.setData("inviterCode", res.user.inviterCode);
									uni.switchTab({
										url: '/pages/index/index'
									})
								} else {
									uni.navigateTo({
										url: '/pages/login/bind'
									});
								}
							});
						}
					}
				} else {
					uni.showToast({
						title: '请同意隐私政策和用户服务协议',
						icon: 'none'
					})
				}
			},
			// 注册
			register() {
				if (this.checked) {
					uni.navigateTo({
						url: '/pages/login/loginPhone'
					});
				} else {
					uni.showToast({
						title: '请同意隐私政策和用户服务协议',
						icon: 'none'
					})
				}

			},
			// 忘记密码
			forget() {
				uni.navigateTo({
					url: '/pages/login/forgetPwd'
				});
			},
			inputChange(e) {
				const key = e.currentTarget.dataset.key;
				this[key] = e.detail.value;
			},
			radioChange(e) {
				console.log(e);
			},
			//第一授权获取用户信息===》按钮触发
			wxGetUserInfo(e) {
				let that = this;
				if (this.checked) {


					wx.getUserProfile({
						desc: '业务需要',
						success: infoRes => {
							console.log("infoRes.encryptedData__________:" + JSON.stringify(infoRes
								.userInfo))
							let nickName = infoRes.userInfo.nickName; //昵称
							let avatarUrl = infoRes.userInfo.avatarUrl; //头像
							let sex = infoRes.userInfo.gender; //头像
							try {
								that.login(nickName, avatarUrl, sex);
							} catch (e) {}
						}
					})
				} else {
					uni.showToast({
						title: '请同意隐私政策和用户服务协议',
						icon: 'none'
					})
				}
			},
			//登录
			login(nickName, avatarUrl, sex) {
				let that = this;
				// 1.wx获取登录用户code
				uni.login({
					provider: 'weixin',
					success: function(loginRes) {
						let data = {
							code: loginRes.code
						}
						that.$Request.get('/app/Login/wxLogin', data).then(res => {
							if (res.code == 0 && res.data) {
								uni.setStorageSync('openId', res.data.open_id)
								uni.setStorageSync('unionId', res.data.unionId)
								that.sessionkey = res.data.session_key;

								let invitationCode = '';
								if (uni.getStorageSync('invitation')) {
									invitationCode = uni.getStorageSync('invitation')
								}
								let sendData = {
									openId: uni.getStorageSync('openId'),
									unionId: uni.getStorageSync('unionId'),
									userName: nickName,
									avatar: avatarUrl,
									sex: sex, //性别
									inviterCode: invitationCode //别人登录进来携带你的邀请码
								};
								that.sendDataList = sendData;
								that.flag = res.data.flag;
								if (that.flag == '1') {
									that.weixinPhone = true;
								} else {
									that.getWeixinInfo(sendData);
								}
							} else {
								uni.showToast({
									icon: 'none',
									title: res.msg,
									duration: 2000
								});
							}
						})

					}
				});
			},
			//小程序微信登录后获取手机号
			getPhoneNumber: function(e) {
				if (e.detail.errMsg == 'getPhoneNumber:fail user deny') {
					console.log('用户拒绝提供手机号');
				} else {
					console.log('用户同意提供手机号');
					this.setPhoneByInsert(e.detail.encryptedData, e.detail.iv);
				}
			},
			//小程序微信登录后获取手机号
			setPhoneByInsert(decryptData, iv) {
				let data = {
					decryptData: decryptData,
					key: this.sessionkey,
					iv: iv
				};

				this.$Request.postJson('/app/Login/selectPhone', data).then(res => {
					if (res.code == 0) {
						this.phone = res.data.phoneNumber;
						this.getWeixinInfo(this.sendDataList);
					} else {
						uni.showToast({
							title: res.msg,
							duration: 2000
						});
					}
				})
			},
			countDown() {
				const {
					count
				} = this;
				if (count === 1) {
					this.count = 60;
					this.sending = false;
					this.sendTime = '获取验证码'
				} else {
					this.count = count - 1;
					this.sending = true;
					this.sendTime = count - 1 + '秒后重新获取';
					setTimeout(this.countDown.bind(this), 1000);
				}
			},
			sendMsg() {
				const {
					mobile
				} = this;
				console.log(mobile)
				if (!mobile) {
					uni.showToast({
						title: '请输入手机号',
						icon: 'none',
						duration: 1000
					})
				} else if (mobile.length !== 11) {
					uni.showToast({
						title: '请输入正确的手机号',
						icon: 'none',
						duration: 1000
					})
				} else {
					uni.showLoading({
						title: '正在发送验证码...'
					})
					this.$Request.get('/app/Login/sendMsg/' + mobile + '/login').then(res => {
						// this.$Request.getT('/appLogin/sendMsg/' + mobile + '/bind').then(res => {
						if (res.code === 0) {
							this.sending = true;
							uni.showToast({
								title: '验证码发送成功请注意查收',
								icon: 'none',
								duration: 1000
							})
							this.countDown();
							uni.hideLoading();
						} else {
							uni.hideLoading();
							uni.showModal({
								showCancel: false,
								title: '短信发送失败',
								content: res.msg ? res.msg : '请一分钟后再获取验证码'
							});
						}
					});
				}
			},
			toLogin() {
				// this.$queue.loginClear();
				// let openid = this.$queue.getData('openid');
				let openid = uni.getStorageSync('openId')
				const {
					mobile,
					code
				} = this;
				if (!mobile) {
					// this.$queue.showToast('请输入手机号');
					uni.showToast({
						title: '请输入手机号',
						icon: 'none',
						duration: 1000
					})
				} else if (mobile.length != 11) {
					// this.$queue.showToast('请输入手机号');
					uni.showToast({
						title: '请输入正确的手机号',
						icon: 'none',
						duration: 1000
					})
				} else if (!code) {
					// this.$queue.showToast('请输入密码');
					uni.showToast({
						title: '请输入密码',
						icon: 'none',
						duration: 1000
					})
				} else {
					uni.showLoading({
						title: '正在登录中...',
					})

					this.$Request.post('/app/Login/registerCode?password=' + code + '&phone=' + mobile).then(res => {
						if (res.code === 0) {
							uni.setStorageSync('token', res.token)
							uni.setStorageSync('userId', res.user.userId)
							uni.setStorageSync('userName', res.user.userName)
							uni.setStorageSync('avatar', res.user.avatar ? res.user.avatar :
								'../../static/images/logo.png')
							uni.setStorageSync('phone', res.user.phone)
							uni.setStorageSync('invitationCode', res.user.invitationCode)
							uni.setStorageSync('sex', res.user.sex)
							uni.setStorageSync('userId', res.user.userId)
							uni.setStorageSync('openId', res.user.openId)
							// this.getWeixinInfo(this.sendDataList);
							uni.showToast({
								title: '登录成功',
								icon: 'none'
							})
							setTimeout(function() {
								uni.switchTab({
									url: '/pages/index/index'
								});
							}, 1000)


						} else {
							uni.hideLoading();
							uni.showToast({
								title: res.msg,
								icon: 'none',
								duration: 1000
							})
						}
					});
				}
			},
			//获取个人信息
			getWeixinInfo(sendData) {
				let that = this;
				uni.showLoading({
					title: '登录中...'
				});
				let postData = {
					openId: sendData.openId, //小程序openId
					unionId: sendData.unionId, //unionId
					userName: sendData.userName, //微信名称
					avatar: sendData.avatar, //头像
					sex: sendData.sex, //性别
					phone: that.phone,
					inviterCode: sendData.inviterCode
				};

				that.$Request.postJson('/app/Login/insertWxUser', postData).then(res => {
					uni.hideLoading();
					if (res.code == 0) {
						uni.setStorageSync('token', res.token)
						uni.setStorageSync('userName', res.user.userName)
						uni.setStorageSync('avatar', res.user.avatar)
						uni.setStorageSync('phone', res.user.phone)
						uni.setStorageSync('invitationCode', res.user.invitationCode)
						uni.setStorageSync('sex', res.user.sex)
						uni.setStorageSync('userId', res.user.userId)
						uni.setStorageSync('openId', res.user.openId)
						let data = {
							userId: uni.getStorageSync('userId')
						}


						that.$Request.get('/app/UserVip/selectUserVip', data).then(res => {
							if (res.code == 0 && res.data && res.data.isVip == 2) {
								uni.setStorageSync('isVIP', true)
							} else {
								uni.setStorageSync('isVIP', false)
							}
						})
						uni.navigateBack();
					} else {
						uni.showModal({
							showCancel: false,
							title: '登录失败',
							content: res.msg,
						});
					}
				})
			},
		}
	};
</script>

<style lang="scss">
	.headers {
		text-align: center;
	}

	.headers>image {
		width: 400upx;
		height: 400upx;
	}

	.footer {
		// padding-left: 150rpx;
		margin-top: 32upx;
		font-size: 24upx;
		color: #666666;
		// text-align: center;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	page {
		background: #fff;
	}

	.send-msg {
		border-radius: 30px;
		color: black;
		background: white;
		height: 30px;
		font-size: 14px;
		line-height: 30px;
	}

	.container {
		top: 0;
		padding-top: 32upx;
		position: relative;
		width: 100%;
		height: 100%;
		overflow: hidden;
		background: #fff;

		.mp_wxBox {
			.headers {
				margin: 35% auto 50rpx;
				text-align: center;
				border-radius: 60rpx;
				width: 650rpx;
				height: 300rpx;
				line-height: 450rpx;

				image {
					width: 300rpx;
					height: 300rpx;
				}
			}

			.content {
				text-align: center;
			}

			text {
				display: block;
				color: #9d9d9d;
				margin-top: 40rpx;
			}

			.bottom {
				line-height: 80upx;
				border-radius: 80upx;
				margin: 70rpx 50rpx;
				height: 80upx;
				font-size: 35rpx;
			}
		}
	}

	.wrapper {
		position: relative;
		z-index: 90;
		background: #fff;
		padding-bottom: 20px;
	}

	.input-content {
		padding: 0 20px;
	}


	.confirm-btn {
		width: 300px;
		height: 42px;
		line-height: 42px;
		border-radius: 30px;
		margin-top: 40px;
		// background: linear-gradient(to left, #3f5ecb 0, #5074FF 100%);
		background: #ffe121;
		color: #333;
		// font-size: $font-lg;

		&:after {
			border-radius: 60px;
		}
	}
</style>