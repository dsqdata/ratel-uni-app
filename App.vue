<script>
	import {
		mapState,
		mapMutations
	} from 'vuex'
	export default {
		onLaunch: function() {
			let uniIdToken = uni.getStorageSync('uniIdToken')
			if (uniIdToken) {
				this.login(uni.getStorageSync('username'))
			}
			console.log('App Launch');
		},
		onShow: function() {
			console.log('App Show');
		},
		onHide: function() {
			console.log('App Hide');
		},
		methods: {
			...mapMutations(['login']),
		}
	}
</script>

<style>
	/* 解决头条小程序组件内引入字体不生效的问题 */
	/* #ifdef MP-TOUTIAO */
	@font-face {
		font-family: uniicons;
		src: url('/static/uni.ttf');
	}

	/* #endif */
	
	/* 头条小程序需要把 iconfont 样式放到组件外 */
	@import "components/ra-icon/ra-icon.css";
	
	/*每个页面公共css */
	page {
		min-height: 100%;
		display: flex;
		font-size: 16px;
	}
	
	/* #ifdef MP-BAIDU */
	page {
		width: 100%;
		height: 100%;
		display: block;
	}
	
	swan-template {
		width: 100%;
		min-height: 100%;
		display: flex;
	}
	
	/* 原生组件模式下需要注意组件外部样式 */
	custom-component {
		width: 100%;
		min-height: 100%;
		display: flex;
	}
	/* #endif */
	
	/* #ifdef MP-ALIPAY */
	page {
		min-height: 100vh;
	}
	/* #endif */
	
	/* 原生组件模式下需要注意组件外部样式 */
	m-input {
		width: 100%;
		/* min-height: 100%; */
		display: flex;
		flex: 1;
	}
	
	.content {
		display: flex;
		flex: 1;
		flex-direction: column;
		background-color: #efeff4;
	}

</style>
