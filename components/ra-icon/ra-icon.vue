<template>
	<view class="ra-icon" :class="['ra-icon-'+type]" @click="onClick()"></view>
</template>

<script>
	export default {
		props: {
			/**
			 * 图标类型
			 */
			type: String
		},
		methods: {
			onClick() {
				this.$emit('click')
			}
		}
	}
</script>

<style>
	@import "./ra-icon.css";
</style>
