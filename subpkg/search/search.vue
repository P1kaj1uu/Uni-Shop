<template>
	<view>
		<!-- 搜索框区域 -->
		<view class="search-box">
			<uni-search-bar :radius="100" cancelButton="none" @input="input" placeholder="请输入搜索内容"></uni-search-bar>
		</view>
		<!-- 搜索建议列表 -->
		<view class="sugg-list" v-if="searchResults.length !== 0">
			<view class="sugg-item" v-for="(item, index) in searchResults" :key="index" @click="gotoDetail(item.goods_id)">
				<view class="goods-name">{{item.goods_name}}</view>
				<uni-icons type="forward" size="16"></uni-icons>
			</view>
		</view>
		<!-- 搜索历史 -->
		<view class="history-box" v-else>
			<!-- 标题区域 -->
			<view class="history-title">
				<text>搜索历史</text>
				<uni-icons type="trash" size="17" @click="cleanHistory"></uni-icons>
			</view>
			<!-- 列表区域 -->
			<view class="history-list">
				<uni-tag :text="item2" v-for="(item2, index2) in historys" :key="index2" @click="gotoGoodsList(item2)"></uni-tag>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				// 延时器的timerId
				timer: null,
				// 搜索关键词
				kw: '',
				// 用于存储搜索结果的列表数据
				searchResults: [],
				// 用于存储搜索关键词的历史记录
				historyList: []
			}
		},
		methods: {
			// 当搜索框value值改变时触发该事件
			input(e) {
				// 清除timer对应的延时器
				clearTimeout(this.timer)
				// 重新启动一个延时器，并把timerId赋值给this.timer
				this.timer = setTimeout(() => {
					// 如果500毫秒内，没有触发新的输入事件，则为搜索关键词赋值
					this.kw = e.valueOf()
					// 根据关键词，查询搜索建议列表
					this.getSearchList()
					// 获取输入的最新的内容
					console.log(e.valueOf())
				}, 500)
			},
			// 根据搜索关键词，获取搜索商品建议列表数据的方法
			async getSearchList() {
				// 判断关键词是否为空
				if (this.kw.trim() === '') {
					this.searchResults = []
					return
				}
				// 发起网络请求
				const { data: res } = await uni.$http.get('/api/public/v1/goods/qsearch', { query: this.kw })
				// 请求失败时
				if (res.meta.status !== 200) return uni.$showMsg()
				// 请求成功时
				this.searchResults = res.message
				// 保存搜索关键词
				this.saveSearchHistory()
				console.log('搜索', res.message)
			},
			// 点击搜索建议的item项，跳转到商品详情页面
			gotoDetail(goods_id) {
				uni.navigateTo({
					// 指定详情页面的URL地址，并传递goods_id参数
					url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods_id
				})
			},
			// 保存搜索关键词的方法
			saveSearchHistory() {
				// 将Array数组转化为Set对象
				const set = new Set(this.historyList)
				// 调用Set对象的delete方法，移除对应的元素
				set.delete(this.kw)
				// 调用Set对象的add方法，向Set中添加元素
				set.add(this.kw)
				// 将Set对象转化为Array数组
				this.historyList = Array.from(set)
				// 将搜索历史记录持久化存储到本地
				uni.setStorageSync('kw', JSON.stringify(this.historyList))
			},
			// 点击清空搜索历史记录
			cleanHistory() {
				// 清空data中保存的搜索历史
				this.historyList = []
				// 清空本地存储中记录的搜索历史
				uni.setStorageSync('kw', '[]')
			},
			// 点击跳转到商品列表页面
			gotoGoodsList(kw) {
				uni.navigateTo({
					url: '/subpkg/goods_list/goods_list?query=' + kw
				})
			}
		},
		computed: {
			historys() {
				return [...this.historyList].reverse()
			}
		},
		onLoad() {
			// 加载本地存储的搜索历史记录
			this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
		}
	}
</script>

<style lang="scss">
.search-box {
	position: sticky;
	top: 0;
	z-index: 999;
}

.sugg-list {
	padding: 0 5px;
	
	.sugg-item {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 13px 0;
		border-bottom: 1px solid #efefef;
		font-size: 12px;
		
		.goods-name {
			// 文字不允许换行（单行文本）
			white-space: nowrap;
			// 溢出部分隐藏
			overflow: hidden;
			// 文本溢出后，使用...代替
			text-overflow: ellipsis;
			margin-right: 3px;
		}
	}
}

.history-box {
  padding: 0 5px;

  .history-title {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 40px;
    font-size: 13px;
    border-bottom: 1px solid #efefef;
  }

  .history-list {
    display: flex;
    flex-wrap: wrap;

    .uni-tag {
			display: block;
      margin-top: 5px;
      margin-right: 5px;
    }
  }
}
</style>
