<template>
	<view>
		<view class="goods-list">
			<view v-for="(item, index) in goodsList" :key="index" @click="gotoDetail(item)">
				<!-- 为my-goods组件动态绑定goods属性的值 -->
				<my-goods :goods="item"></my-goods>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				// 请求参数对象
				queryObj: {
					// 查询关键词
					query: '',
					// 商品分类id
					cid: '',
					// 页码值
					pagenum: 1,
					// 每页显示多少条数据
					pagesize: 10
				},
				// 用于存储商品列表的数据
				goodsList: [],
				// 总数量，用来实现分页
				total: 0,
				// 是否正在请求数据，通过节流阀防止发起额外的请求
				isloading: false
			}
		},
		onLoad(options) {
			// 将页面跳转时携带的参数，转存到queryObj对象中
			this.queryObj.query = options.query || ''
			this.queryObj.cid = options.cid || ''
			// 调用获取商品列表数据的方法
			this.getGoodsList()
		},
		methods: {
			// 获取商品列表数据的方法
			async getGoodsList(cb) {
				// 打开节流阀
				this.isloading = true
				// 发起网络请求
				const { data: res } = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
				// 关闭节流阀
				this.isloading = false
				// 只要数据请求完毕，就立即按需调用cb回调函数
				cb && cb()
				// 请求失败时
				if (res.meta.status !== 200) return uni.$showMsg()
				// 请求成功时
				this.goodsList = [...this.goodsList, ...res.message.goods]
				this.total = res.message.total
				console.log('商品列表', res.message)
			},
			// 点击跳转到商品详情页面
			gotoDetail(item) {
				uni.navigateTo({
					url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
				})
			}
		},
		// 上拉触底的事件
		onReachBottom() {
			// 判断是否还有下一页数据
			if (this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg('到底啦...')
			// 判断是否正在请求其它数据，如果是，则不发起额外的请求
			if (this.isloading) return
			// 让页码值自增+1
			this.queryObj.pagenum += 1
			// 重新获取列表数据
			this.getGoodsList()
		},
		// 下拉刷新的事件
		onPullDownRefresh() {
			// 重置关键数据
			this.queryObj.pagenum = 1
			this.total = 0
			this.isloading = false
			this.goodsList = []
			// 重新发起请求
			this.getGoodsList(() => uni.stopPullDownRefresh())
		}
	}
</script>

<style lang="scss">

</style>
