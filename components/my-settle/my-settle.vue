<template>
	<view class="my-settle-container">
		<!-- 全选区域 -->
		<label class="radio" @click="changeAllState">
		  <radio color="#C00000" :checked="isFullCheck" /><text>全选</text>
		</label>
		<!-- 合计区域 -->
		<view class="amount-box">
		  合计:<text class="amount">￥{{checkedGoodsAmount}}</text>
		</view>
		<!-- 结算区域 -->
		<view class="btn-settle">结算({{checkedCount}})</view>
	</view>
</template>

<script>
	import { mapGetters, mapMutations } from 'vuex'
	
	export default {
		name:"my-settle",
		data() {
			return {
				
			}
		},
		computed: {
			...mapGetters('m_cart', ['checkedCount', 'total', 'checkedGoodsAmount']),
			// 是否全选
			isFullCheck() {
				return this.total === this.checkedCount
			}
		},
		methods: {
			// 使用mapMutations辅助函数，把m_cart模块提供的updateAllGoodsState方法映射到当前组件中使用
			...mapMutations('m_cart', ['updateAllGoodsState']),
			// 全选区域按钮区域点击事件
			changeAllState() {
				// 修改购物车中所有商品的选中状态
				// !this.isFullCheck 表示：当前全选按钮的状态取反之后，就是最新的勾选状态
				this.updateAllGoodsState(!this.isFullCheck)
			}
		}
	}
</script>

<style lang="scss">
.my-settle-container {
	display: flex;
	justify-content: space-between;
	align-items: center;
	position: fixed;
	padding-left: 5px;
	bottom: 0;
	left: 0;
	width: 100%;
	height: 50px;
	font-size: 14px;
	background-color: white;
	
	.radio {
		display: flex;
		align-items: center;
	}
	
	.amount {
		color: #C00000;
	}
	
	.btn-settle {
		padding: 0 10px;
		min-width: 100px;
		height: 50px;
		line-height: 50px;
		text-align: center;
		color: white;
		background-color: #C00000;
	}
}
</style>
