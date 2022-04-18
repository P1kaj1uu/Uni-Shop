<template>
	<view class="goods-item">
		<!-- 商品左侧图片区域 -->
		<view class="goods-item-left">
			<radio :checked="goods.goods_state" color="#C00000" @click="radioClickHandler" v-if="showRadio"></radio>
			<image :src="goods.goods_small_logo || defaultPic" class="goods-pic"></image>
		</view>
		<!-- 商品右侧信息区域 -->
		<view class="goods-item-right">
			<!-- 商品标题 -->
			<view class="goods-name">{{goods.goods_name}}</view>
			<view class="goods-info-box">
				<!-- 商品价格 -->
				<view class="goods-price">￥{{goods.goods_price | tofixed}}</view>
				<!-- 商品数量 -->
				<uni-number-box :min="1" :value="goods.goods_count" v-if="showNum" @change="numChangeHandler"></uni-number-box>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		name:"my-goods",
		// 定义props属性，用来接收外界传递到当前组件的数据
		props: {
			// 商品的信息对象
			goods: {
				type: Object,
				default: {}
			},
			// 是否展示图片左侧的radio
			showRadio: {
				type: Boolean,
				// 如果外界没有指定showRadio属性的值，则默认不展示radio组件
				default: false
			},
			// 是否展示价格右侧的NumberBox组件
			showNum: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				// 默认的空图片
				defaultPic: 'https://img3.doubanio.com/f/movie/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/movie/celebrity-default-medium.png'
			}
		},
		filters: {
			// 把数字处理为带两位小数点的数字
			tofixed(num) {
				return Number(num).toFixed(2)
			}
		},
		methods: {
			// radio组件的点击事件处理函数
			radioClickHandler() {
				// 通过 this.$emit() 触发外界通过 @ 绑定的 radio-change 事件
				// 同时把商品的id和勾选状态作为参数传递给radio-change事件处理函数
				this.$emit('radio-change', {
					// 商品的id
					goods_id: this.goods.goods_id,
					// 商品最新的勾选状态
					goods_state: !this.goods.goods_state
				})
			},
			// NumberBox组件的change事件处理函数
			numChangeHandler(val) {
				// 通过this.$emit()触发外界通过@绑定的num-change事件
				this.$emit('num-change', {
					// 商品的id
					goods_id: this.goods.goods_id,
					// 商品的最新数量
					goods_count: +val
				})
			}
		}
	}
</script>

<style lang="scss">
.goods-item {
	display: flex;
	padding: 10px 5px;
	border-bottom: 1px solid #F0F0F0;
	width: 750rpx;
	box-sizing: border-box;
	
	.goods-item-left {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-right: 5px;
		
		.goods-pic {
			display: block;
			width: 100px;
			height: 100px;
		}
	}
	
	.goods-item-right {
		display: flex;
		flex: 1;
		flex-direction: column;
		justify-content: space-between;
		
		.goods-name {
			font-size: 13px;
		}
		
		.goods-info-box {
			display: flex;
			align-items: center;
			justify-content: space-between;
		}
		
		.goods-price {
			font-size: 16px;
			color: #C00000;
		}
	}
}
</style>
