<template>
	<view>
		<!-- 使用自定义的搜索组件 -->
		<my-search @click="gotoSearch"></my-search>
		<view class="scroll-view-container">
			<!-- 左侧的滚动视图区域 -->
			<scroll-view class="left-scroll-view" :style="{height: wh + 'px'}" scroll-y="true">
				<block v-for="(item, index) in cateList" :key="index">
					<view :class="['left-scroll-view-item', index === active ? 'active' : '']" @click="activeChanged(index)">{{item.cat_name}}</view>
				</block>
			</scroll-view>
			<!-- 右侧的滚动视图区域 -->
			<scroll-view class="right-scroll-view" :style="{height: wh + 'px'}" scroll-y="true" :scroll-top="scrollTop">
				<view class="cate-lv2" v-for="(item2, index2) in cateLevel" :key="index2">
					<!-- 二级分类的标题 -->
				  <view class="cate-lv2-title">/ {{item2.cat_name}} /</view>
					<!-- 动态渲染三级分类的列表数据 -->
					<view class="cate-lv3-list">
						<!-- 三级分类的item项 -->
						<view class="cate-lv3-item" v-for="(item3, index3) in item2.children" :key="index3" @click="gotoGoodsList(item3)">
							<!-- 图片 -->
							<image :src="item3.cat_icon"></image>
							<!-- 文本 -->
							<text>{{item3.cat_name}}</text>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import badgeMix from '../../mixins/tabbar-badge.js'
	
	export default {
		mixins: [badgeMix],
		data() {
			return {
				// 用于转存调用getSystemInfoSync方法获取的屏幕可用高度
				wh: 0,
				// 用于存储左侧分类列表的数据
				cateList: [],
				// 设置激活项，默认第一项被激活选中
				active: 0,
				// 用于存储右侧二级分类列表的数据
				cateLevel: [],
				// 滚动条距离顶部的距离
				scrollTop: 0
			}
		},
		onLoad() {
			// 获取设备信息
			const sysInfo = uni.getSystemInfoSync()
			console.log(sysInfo)
			// 可用高度 = 屏幕高度 - navigationBar高度 - tabBar高度 - 自定义的search组件高度
			this.wh = sysInfo.windowHeight - 50
			// 调用获取分类列表数据的方法
			this.getCateList()
		},
		methods: {
			// 获取分类数据的方法
			async getCateList() {
				// 发起网络请求
				const { data: res } = await uni.$http.get('/api/public/v1/categories')
				// 请求失败时
				if (res.meta.status !== 200) return uni.$showMsg()
				// 请求成功
				// 为左侧分类数据列表赋值
				this.cateList = res.message
				// 为右侧二级分类数据列表赋值
				this.cateLevel = res.message[0].children
				console.log('分类', res.message)
			},
			// 点击切换激活项
			activeChanged(index) {
				this.active = index
				// 为二级分类列表重新赋值
				this.cateLevel = this.cateList[index].children
				// 让scrollTop的值在0与1之间切换
				this.scrollTop = this.scrollTop === 0 ? 1 : 0
			},
			// 点击三级分类项跳转到商品列表页面
			gotoGoodsList(item3) {
				uni.navigateTo({
					url: '/subpkg/goods_list/goods_list?cid=' + item3.cat_id
				})
			},
			// 跳转到分包中的搜索页面
			gotoSearch() {
				uni.navigateTo({
					url: '/subpkg/search/search'
				})
			}
		}
	}
</script>

<style lang="scss">
.scroll-view-container {
	display: flex;
	
	.left-scroll-view {
		width: 120px;
		
		.left-scroll-view-item {
			background-color: #F7F7F7;
			line-height: 60px;
			text-align: center;
			font-size: 12px;
			
			// 激活项的样式
			&.active {
				position: relative;
				background-color: #FFFFFF;
				
				// 渲染激活项左侧的红色指示边线
				&::before {
					content: ' ';
					display: block;
					width: 3px;
					height: 30px;
					background-color: #C00000;
					position: absolute;
					left: 0;
					top: 50%;
					transform: translateY(-50%);
				}
			}
		}
	}
}

.cate-lv2-title {
  font-size: 12px;
  font-weight: bold;
  text-align: center;
  padding: 15px 0;
}

.cate-lv3-list {
  display: flex;
  flex-wrap: wrap;

  .cate-lv3-item {
    width: 33.33%;
    margin-bottom: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;

    image {
      width: 60px;
      height: 60px;
    }

    text {
      font-size: 12px;
    }
  }
}
</style>
