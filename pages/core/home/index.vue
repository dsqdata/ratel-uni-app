<template>
	<view>
		<button type="primary" class="primary" @tap="outLogin">注销</button>
		<button type="primary" class="primary" @tap="myInfo">我的信息</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {

			}
		},

		onLoad() {
			let ratelUser = uni.getStorageSync('ratelUser')
			if (ratelUser) {

			} else {
				this.guideToLogin()
			}
		},
		methods: {
			guideToLogin() {
				uni.reLaunch({
					url: '../login/index',
				});
			},
			outLogin() {
				uni.removeStorageSync("ratelUser")
				uni.reLaunch({
					url: './index',
				});
			},
			myInfo(){
				uni.redirectTo({
					url:"../my/index"
				})
			}
		}
	}
</script>

<style>

</style>
