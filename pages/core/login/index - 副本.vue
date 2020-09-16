<template>
	<view class="content">
		<view class="img-view">
			<image src="../../../static/img/bg.png"></image>
		</view>

		<view class="content-body">
			<view class="login-type">
				<view v-for="(item,index) in loginTypeList" :key="index" @click="loginType = index" :class="{act: loginType === index}"
				 class="login-type-btn">{{item}}</view>
			</view>
			<view class="" v-if="loginType === 0">
				<view class="ratel-input-row ratel-border-bottom">
					<ra-input class="ra-input" type="text" clearable focus v-model="username" placeholder="请输入账号"></ra-input>
				</view>
				<view class="ratel-input-row ratel-border-bottom">
					<ra-input type="password" displayable v-model="password" placeholder="请输入密码"></ra-input>
				</view>
			</view>
			<view class="" v-else>
				<view class="ratel-input-row ratel-border-bottom">
					<ra-input class="ra-input" type="text" clearable focus v-model="mobile" placeholder="请输入手机号码"></ra-input>
				</view>
				<view class="ratel-input-row ratel-border-bottom">
					<ra-input type="text" v-model="code" placeholder="请输入验证码"></ra-input>
					<view class="send-code-btn" @click="sendSmsCode">{{codeDuration ? codeDuration + 's' : '发送验证码' }}</view>
				</view>
			</view>
			<view class="ratel-btn-row">
				<button type="primary" class="primary ratel-button" @tap="bindLogin">登录</button>
			</view>
			<view class="ratel-action-row">
				<navigator url="../reg/reg">注册账号</navigator>
				<text style="padding: 0 10rpx;">|</text> 
				<navigator url="../pwd/pwd">忘记密码</navigator>
			</view>
			<view class="oauth-row" v-if="hasProvider" v-bind:style="{top: positionTop + 'px'}">
				<view class="oauth-image" v-for="provider in providerList" :key="provider.value">
					<image :src="provider.image" @tap="oauth(provider.value)"></image>
					<!-- #ifdef MP-WEIXIN -->
					<button v-if="!isDevtools" open-type="getUserInfo" @getuserinfo="getUserInfo"></button>
					<!-- #endif -->
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import login from '../../../service/login.js';
	import {
		encrypt
	} from '../../../core/net/RsaEncrypt.js'
	import {
		mapState,
		mapMutations
	} from 'vuex'
	import raInput from '../../../components/ra-input/ra-input.vue'
	import raIcon from '../../../components/ra-icon/ra-icon.vue'

	export default {
		components: {
			raInput,
			raIcon
		},
		data() {
			return {
				loginType: 0,
				loginTypeList: ['密码登录', '免密登录'],
				mobile: '',
				code: '',
				providerList: [],
				hasProvider: false,
				username: '',
				password: '',
				positionTop: 0,
				isDevtools: false,
				codeDuration: 0
			}
		},
		computed: mapState(['forcedLogin']),
		methods: {
			...mapMutations(['login']),
			initProvider() {
				const filters = ['weixin', 'qq', 'sinaweibo'];
				uni.getProvider({
					service: 'oauth',
					success: (res) => {
						if (res.provider && res.provider.length) {
							for (let i = 0; i < res.provider.length; i++) {
								if (~filters.indexOf(res.provider[i])) {
									this.providerList.push({
										value: res.provider[i],
										image: '../../static/img/' + res.provider[i] + '.png'
									});
								}
							}
							// this.hasProvider = true;
						}
					},
					fail: (err) => {
						console.error('获取服务供应商失败：' + JSON.stringify(err));
					}
				});
			},
			initPosition() {
				/**
				 * 使用 absolute 定位，并且设置 bottom 值进行定位。软键盘弹出时，底部会因为窗口变化而被顶上来。
				 * 反向使用 top 进行定位，可以避免此问题。
				 */
				this.positionTop = uni.getSystemInfoSync().windowHeight - 100;
			},
			sendSmsCode() {
				if (this.codeDuration) {
					uni.showModal({
						content: `请在${this.codeDuration}秒后重试`,
						showCancel: false
					})
				}
				if (!/^1\d{10}$/.test(this.mobile)) {
					uni.showModal({
						content: '手机号码填写错误',
						showCancel: false
					})
					return
				}
				uniCloud.callFunction({
					name: 'user-center',
					data: {
						action: 'sendSmsCode',
						params: {
							mobile: this.mobile
						}
					},
					success: (e) => {
						if (e.result.code == 0) {
							uni.showModal({
								content: '验证码发送成功，请注意查收',
								showCancel: false
							})
							this.codeDuration = 60
							this.codeInterVal = setInterval(() => {
								this.codeDuration--
								if (this.codeDuration === 0) {
									if (this.codeInterVal) {
										clearInterval(this.codeInterVal)
										this.codeInterVal = null
									}
								}
							}, 1000)
						} else {
							uni.showModal({
								content: '验证码发送失败：' + e.result.msg,
								showCancel: false
							})
						}

					},
					fail(e) {
						uni.showModal({
							content: '验证码发送失败',
							showCancel: false
						})
					}
				})
			},
			loginByPwd() {
				/**
				 * 客户端对账号信息进行一些必要的校验。
				 * 实际开发中，根据业务需要进行处理，这里仅做示例。
				 */
				if (this.username.length < 3) {
					uni.showToast({
						icon: 'none',
						title: '账号最短为 3 个字符'
					});
					return;
				}
				if (this.password.length < 6) {
					uni.showToast({
						icon: 'none',
						title: '密码最短为 6 个字符'
					});
					return;
				}
				const data = {
					username: this.username,
					password: encrypt(this.password)
				};
				let _self = this;

				this.$request.post({
					url: 'auth/login',
					data: data,
					loadingTip: '正在登录...',
					success: res => {
						uni.setStorageSync('ratelUsername', res.user.username)
						uni.setStorageSync('ratelToken', res.token)
						uni.setStorageSync('ratelUser', res.user)
						// uni.setStorageSync('login_type', 'online')
						_self.toMain(res.user.username);
					},
					fail: res => {
						console.log(res)
						uni.showModal({
							content: !!res.data.message ? res.data.message : '系统错误，请联系管理员！',
							showCancel: false
						})
					},
					complete: res => {

					}
				});
			},
			loginBySms() {
				if (!/^1\d{10}$/.test(this.mobile)) {
					uni.showModal({
						content: '手机号码填写错误',
						showCancel: false
					})
					return
				}
				if (!/^\d{6}$/.test(this.code)) {
					uni.showModal({
						title: '验证码为6位纯数字',
						showCancel: false
					});
					return;
				}
				let _self = this;

				uniCloud.callFunction({
					name: 'user-center',
					data: {
						action: 'loginBySms',
						params: {
							mobile: this.mobile,
							code: this.code
						}
					},
					success: (e) => {

						console.log('login success', e);

						if (e.result.code == 0) {
							const username = e.result.username || '新用户'
							uni.setStorageSync('uniIdToken', e.result.token)
							uni.setStorageSync('username', username)
							uni.setStorageSync('login_type', 'online')
							_self.toMain(username);
						} else {
							uni.showModal({
								content: e.result.msg,
								showCancel: false
							})
							console.log('登录失败', e);
						}

					},
					fail(e) {
						uni.showModal({
							content: JSON.stringify(e),
							showCancel: false
						})
					}
				})
			},
			bindLogin() {
				switch (this.loginType) {
					case 0:
						this.loginByPwd()
						break;
					case 1:
						this.loginBySms()
						break;
					default:
						break;
				}
			},
			oauth(value) {
				console.log('三方登录只演示登录api能力，暂未关联云端数据');
				uni.login({
					provider: value,
					success: (res) => {
						uni.getUserInfo({
							provider: value,
							success: (infoRes) => {
								/**
								 * 实际开发中，获取用户信息后，需要将信息上报至服务端。
								 * 服务端可以用 userInfo.openId 作为用户的唯一标识新增或绑定用户信息。
								 */
								this.loginLocal(infoRes.userInfo.nickName);
							},
							fail() {
								uni.showToast({
									icon: 'none',
									title: '登陆失败'
								});
							}
						});
					},
					fail: (err) => {
						console.error('授权登录失败：' + JSON.stringify(err));
					}
				});
			},
			getUserInfo({
				detail
			}) {
				console.log('三方登录只演示登录api能力，暂未关联云端数据');
				if (detail.userInfo) {
					this.loginLocal(detail.userInfo.nickName);
				} else {
					uni.showToast({
						icon: 'none',
						title: '登陆失败'
					});
				}
			},
			loginLocal(nickName) {
				uni.setStorageSync('login_type', 'local')
				uni.setStorageSync('username', nickName)
				this.toMain(nickName);
			},
			toMain(userName) {
				this.login(userName);
				/**
				 * 强制登录时使用reLaunch方式跳转过来
				 * 返回首页也使用reLaunch方式
				 */
				if (this.forcedLogin) {
					uni.reLaunch({
						url: '../home/index',
					});
				} else {
					uni.navigateBack();
				}

			}
		},
		onReady() {
			this.initPosition();
			this.initProvider();
			// #ifdef MP-WEIXIN
			this.isDevtools = uni.getSystemInfoSync().platform === 'devtools';
			// #endif
		}
	}
</script>

<style>
	.pages-core-login-index .content {
		background-color: #FFFFFF;
	}

	.content-body {
		padding: 24px;
	}

	image,
	.img-view {
		width: 750rpx;
		height: 600rpx;
	}

	.ratel-input-row {
		margin-top: 10px;
		padding: 10px 10px 5px 10px;
		display: flex;
		flex-direction: row;
		position: relative;
		    font-size: 14px;
		    line-height: 14px;
	}

	.ratel-input-row .title {
		width: 100px;
		padding-left: 15px;
	}

	.ratel-border-bottom {
		border-bottom: 1px solid #cacaca;
	}

	.ratel-btn-row {
		margin-top: 40px;
	}
	
	.ratel-action-row{
		display: flex;
		flex-direction: row;
		justify-content: center;
		margin-top: 20rpx;
		font-size: 20rpx;
		color: #555555;
	}

	.ratel-button {
		background-color: #57ade8;
		border-radius:50rpx;
		line-height: 2.3;
	}


	.login-type {
		display: flex;
		justify-content: center;
	}

	.login-type-btn {
		line-height: 30px;
		margin: 0px 15px;
	}

	.login-type-btn.act {
		color: #0FAEFF;
		border-bottom: solid 1px #0FAEFF;
	}

	.send-code-btn {
		line-height: 24px;
		font-size: 14px;
		width: 95px;
		text-align: center;
		background-color: #0FAEFF;
		border-radius: 20px;
		color: #FFFFFF;
	}

	.action-row {
		
	}

	.action-row navigator {
		color: #007aff;
		padding: 0 10px;
	}

	.oauth-row {
		display: flex;
		flex-direction: row;
		justify-content: center;
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
	}

	.oauth-image {
		position: relative;
		width: 50px;
		height: 50px;
		border: 1px solid #dddddd;
		border-radius: 50px;
		margin: 0 20px;
		background-color: #ffffff;
	}

	.oauth-image image {
		width: 30px;
		height: 30px;
		margin: 10px;
	}

	.oauth-image button {
		position: absolute;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		opacity: 0;
	}

	.input-group {
		background-color: initial;
	}

	.input-row.border::after {
		background-color: #FFFFFF;
	}
</style>
