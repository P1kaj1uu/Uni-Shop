<template>
	<view class="cart-container" v-if="cart.length !== 0">
		<!-- 收货地址组件 -->
		<my-address></my-address>
		<!-- 购物车商品列表的标题区域 -->
		<view class="cart-title">
			<!-- 左侧的图标 -->
			<uni-icons type="shop" size="18"></uni-icons>
			<!-- 描述文本 -->
			<text class="cart-title-text">购物车</text>
		</view>
		<!-- 购物车商品列表区域 -->
		<uni-swipe-action>
			<block v-for="(goods, index) in cart" :key="index">
				<uni-swipe-action-item :right-options="options" @click="swipeActionClickHandler(goods)">
					<my-goods :goods="goods" :showRadio="true" :showNum="true" @radio-change="radioChangeHandler" @num-change="numberChangeHandler"></my-goods>
				</uni-swipe-action-item>
			</block>
		</uni-swipe-action>
		<!-- 结算组件 -->
		<my-settle></my-settle>
	</view>
	<!-- 空白购物车区域 -->
	<view class="empty-cart" v-else>
	  <image src="../../static/cart_empty@2x.png" class="empty-img"></image>
	  <text class="tip-text">空空如也~</text>
	</view>
</template>

<script>
	import badgeMix from '../../mixins/tabbar-badge.js'
	import { mapState, mapMutations } from 'vuex'
	
	export default {
		mixins: [badgeMix],
		data() {
			return {
				options: [{
					// 显示的文本内容
					text: '删除',
					style: {
						// 按钮的背景颜色
						backgroundColor: '#C00000'
					}
				}]
			}
		},
		methods: {
			...mapMutations('m_cart', ['updateGoodsState', 'updateGoodsCount', 'removeGoodsById']),
			// 商品的勾选状态发生了变化
			radioChangeHandler(e) {
				this.updateGoodsState(e)
			},
			// 商品的数量发生了变化
			numberChangeHandler(e) {
				this.updateGoodsCount(e)
			},
			// 点击了滑动操作按钮
			swipeActionClickHandler(goods) {
				this.removeGoodsById(goods.goods_id)
			}
		},
		computed: {
			// 将m_cart模块中的cart数组映射到当前页面中使用
			...mapState('m_cart', ['cart'])
		}
	}
</script>

<style lang="scss">
.cart-container {
	padding-bottom: 50px;
}

.cart-title {
	display: flex;
	align-items: center;
	padding-left: 5px;
	height: 40px;
	font-size: 14px;
	border-bottom: 1px solid #efefef;
	
	.cart-title-text {
		margin-left: 10px;
	}
}

.empty-cart {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 150px;

  .empty-img {
    width: 90px;
    height: 90px;
  }

  .tip-text {
    font-size: 12px;
    color: gray;
    margin-top: 15px;
  }
}
</style>
