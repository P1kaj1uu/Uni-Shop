<template>
	<view class="goods-detail-container" v-if="goods_info.goods_name">
		<!-- 轮播图区域 -->
		<swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular="true">
			<swiper-item v-for="(item, index) in goods_info.pics" :key="index">
				<image :src="item.pics_big" @click="preview(index)"></image>
			</swiper-item>
		</swiper>
		<!-- 商品信息区域 -->
		<view class="goods-info-box">
			<!-- 商品价格 -->
			<view class="price">￥{{goods_info.goods_price | toFixed}}</view>
			<!-- 信息主体区域 -->
			<view class="goods-info-body">
				<!-- 商品名称 -->
				<view class="goods-name">{{goods_info.goods_name}}</view>
				<!-- 收藏 -->
				<view class="favi">
					<uni-icons type="star" size="18" color="gray"></uni-icons>
					<text>收藏</text>
				</view>
			</view>
			<!-- 运费 -->
			<view class="yf">快递：免运费</view>
		</view>
		<!-- 商品详情信息 -->
		<rich-text :nodes="goods_info.goods_introduce"></rich-text>
		<!-- 商品导航组件 -->
		<view class="goods_nav">
		  <!-- fill 控制右侧按钮的样式 -->
		  <!-- options 左侧按钮的配置项 -->
		  <!-- buttonGroup 右侧按钮的配置项 -->
		  <!-- click 左侧按钮的点击事件处理函数 -->
		  <!-- buttonClick 右侧按钮的点击事件处理函数 -->
		  <uni-goods-nav :fill="true" :options="options" :buttonGroup="buttonGroup" @click="onClick" @buttonClick="buttonClick" />
		</view>
	</view>
</template>

<script>
	import { mapState, mapMutations, mapGetters } from 'vuex'
	
	export default {
		data() {
			return {
				// 用于存储商品详情列表的数据对象
				goods_info: {},
				// 左侧按钮组的配置对象
				options: [{
					icon: 'shop',
					text: '店铺'
				}, {
					icon: 'cart',
					text: '购物车',
					info: 0
				}],
				// 右侧按钮组的配置对象
				buttonGroup: [{
					text: '加入购物车',
					backgroundColor: '#ff0000',
					color: '#fff'
				}, {
					text: '立即购买',
					backgroundColor: '#ffa200',
					color: '#fff'
				}]
			}
		},
		onLoad(options) {
			// 获取商品id
			const goods_id = options.goods_id
			// 调用请求商品详情数据的方法
			this.getGoodsDetail(goods_id)
		},
		methods: {
			// 把m_cart模块中的addToCart方法映射到当前页面使用
			...mapMutations('m_cart', ['addToCart']),
			// 获取商品详情数据的方法
			async getGoodsDetail(goods_id) {
				// 发起网络请求
				const { data: res } = await uni.$http.get('/api/public/v1/goods/detail', { goods_id })
				// 请求失败时
				if (res.meta.status !== 200) return uni.$showMsg()
				// 请求成功时
				// 使用字符串的replace()方法，为img标签添加行内的style样式，从而解决图片底部空白间隙的问题，并将webp的后缀名替换为jpg的后缀名
				res.message.goods_introduce = res.message.goods_introduce.replace(/<img /g, '<img style="display:block;" ').replace(/webp/g, 'jpg')
				this.goods_info = res.message
				console.log('商品详情', res.message)
			},
			// 实现轮播图的预览效果
			preview(index) {
				uni.previewImage({
					// 预览时，默认显示图片的索引
					current: index,
					// 所有图片url地址的数组
					urls: this.goods_info.pics.map(x => x.pics_big)
				})
			},
			// 点击商品导航组件左侧的按钮，跳转到购物车页面
			onClick(e) {
				if (e.content.text === '购物车') {
					uni.switchTab({
						url: '/pages/cart/cart'
					})
				}
			},
			// 右侧按钮的点击事件处理函数
			buttonClick(e) {
				// 判断是否点击了加入购物车按钮
				if (e.content.text === '加入购物车') {
					// 组织一个商品的信息对象
					const goods = {
						// 商品的id
						goods_id: this.goods_info.goods_id,
						// 商品名称
						goods_name: this.goods_info.goods_name,
						// 商品价格
						goods_price: this.goods_info.goods_price,
						// 商品数量
						goods_count: 1,
						// 商品图片
						goods_small_logo: this.goods_info.goods_small_logo,
						// 商品勾选状态
						goods_state: true
					}
					// 通过this调用映射过来的addToCart方法，把商品信息对象存储到购物车中
					this.addToCart(goods)
				}
			}
		},
		filters: {
			toFixed(num) {
				return Number(num).toFixed(2)
			}
		},
		computed: {
			// 把m_cart模块中名称为total的getter映射到当前页面中使用
			...mapGetters('m_cart', ['total'])
		},
		watch: {
			// 监听total值的变化，通过第一个形参得到变化后的新值
			total: {
				// handler属性用来定义侦听器的function处理函数
				handler(newVal) {
					// 通过数组的find()方法，找到购物车按钮的配置对象
					const findResult = this.options.find((x) => x.text === '购物车')
					if (findResult) {
						// 动态为购物车按钮的info属性赋值
						findResult.info = newVal
					}
				},
				// immediate属性用来声明此侦听器，是否在页面初次加载完毕后立即调用
				immediate: true
			}
		}
	}
</script>

<style lang="scss">
swiper {
	height: 700rpx;
	
	image {
		width: 100%;
		height: 100%;
	}
}

.goods-info-box {
	padding: 10px;
	padding-right: 0;
	
	.price {
		margin: 10px 0;
		color: #C00000;
		font-size: 18px;
	}
	
	.goods-info-body {
		display: flex;
		justify-content: space-between;
		
		.goods-name {
			padding-right: 10px;
			font-size: 13px;
		}
		
		.favi {
			display: flex;
			flex-direction: column;
			justify-content: center;
			align-items: center;
			border-left: 1px solid #efefef;
			color: gray;
			font-size: 12px;
			width: 120px;
		}
	}
	
	.yf {
		margin: 10px 0;
		color: gray;
		font-size: 12px;
	}
}

.goods_nav {
	position: fixed;
	bottom: 0;
	left: 0;
	width: 100%;
}
	
.goods-detail-container {
  // 给页面外层的容器，添加50px的内padding
  // 防止页面内容被底部的商品导航组件遮盖
  padding-bottom: 50px;
}
</style>
