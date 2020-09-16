<template>

	<div class="ra-form-group">
		<label class="ra-form-label">{{title}}</label>
		<view :class="isFocus?'ra-input-view ra-input-view-act':'ra-input-view'">

			<input :focus="focus" :type="inputType" :value="value" @input="onInput" class="ra-form-control" :placeholder="placeholder"
			 :password="type==='password'&&!showPassword" @focus="onFocus" @blur="onBlur" />

			<view v-if="clearable&&!displayable&&value.length" class="ra-input-icon">
				<ra-icon color="#1060c0" type="clear" @click="clear"></ra-icon>
			</view>
			<view v-if="displayable" class="ra-input-icon">
				<ra-icon :style="{color:showPassword?'#1060c0':'#cccccc'}" type="eye" @click="display"></ra-icon>
			</view>
		</view>
		<div class="clearfix"></div>
	</div>

</template>

<script>
	import raIcon from '../ra-icon/ra-icon.vue'

	export default {
		components: {
			raIcon
		},
		props: {
			title: String,
			/**
			 * 输入类型
			 */
			type: String,
			/**
			 * 值
			 */
			value: String,
			/**
			 * 占位符
			 */
			placeholder: String,
			/**
			 * 是否显示清除按钮
			 */
			clearable: {
				type: [Boolean, String],
				default: false
			},
			/**
			 * 是否显示密码可见按钮
			 */
			displayable: {
				type: [Boolean, String],
				default: false
			},
			/**
			 * 自动获取焦点
			 */
			focus: {
				type: [Boolean, String],
				default: false
			}
		},
		model: {
			prop: 'value',
			event: 'input'
		},
		data() {
			return {
				/**
				 * 显示密码明文
				 */
				showPassword: false,
				/**
				 * 是否获取焦点
				 */
				isFocus: false
			}
		},
		computed: {
			inputType() {
				const type = this.type
				return type === 'password' ? 'text' : type
			}
		},
		methods: {
			clear() {
				this.$emit('input', '')
			},
			display() {
				this.showPassword = !this.showPassword
			},
			onFocus() {
				this.isFocus = true
			},
			onBlur() {
				this.$nextTick(() => {
					this.isFocus = false
				})
			},
			onInput(e) {
				this.$emit('input', e.detail.value)
			}
		}
	}
</script>

<style>
	.uni-input-placeholder {
		font-size: 12px;
		color: #a0b0d0;
	}

	.ra-input-view {
		display: inline-flex;
		flex-direction: row;
		align-items: center;
		width: 100%;
		flex: 1;
		padding: 0px;
		padding-top: 3px;
		border: 0px solid rgba(24, 28, 33, 0.1);
		border-bottom-width: 1px;
		border-radius: 0 !important;
		border-radius: 0px;
		-webkit-transition: border-color 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
		transition: border-color 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
		-webkit-transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
		transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
		transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
	}

	.ra-input-view-act {
		border: 0px solid #1060c0;
		border-bottom-width: 1px;
	}

	.ra-input-icon {
		width: 20px;
		font-size: 20px;
		line-height: 20px;
		color: #1060c0;
	}

	.ra-form-group {
		margin-bottom: 1rem;
	}

	.ra-form-label {
		display: inline-block;
		box-sizing: border-box;
		margin-bottom: 0;
		font-weight: 500;
		font-size: 14px;
		font-family: "Roboto", -apple-system, BlinkMacSystemFont, "Segoe UI", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
		color: #1060c0;
		text-align: left;
	}


	.ra-form-group input {
		display: block;
		width: 100%;
		height: 28px;
		font-size: 14px;
		font-weight: 400;
		line-height: 1.33;
		color: #4E5155;
		background-color: transparent;
		background-clip: padding-box;
		border: 0px;
		border-radius: 0px;
		-webkit-transition: border-color 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
		transition: border-color 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
		transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
		transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out, -webkit-box-shadow 0.15s ease-in-out;
	}

	.ra-form-group input {
		padding-right: 0;
		padding-left: 0;
		padding-bottom: 0 !important;
	}

	.ra-form-control input:focus {
		color: #4E5155;
		background-color: transparent;
		border-color: #cbc8eb;
		outline: 0;
		-webkit-box-shadow: none;
		box-shadow: none;
	}

	.clearfix::after {
		display: block;
		clear: both;
		content: "";
	}
</style>
