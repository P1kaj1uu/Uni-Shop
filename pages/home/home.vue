<template>
	<view>
		<!-- 使用自定义搜索组件 -->
		<view class="search-box">
			<my-search @click="gotoSearch"></my-search>
		</view>
		<!-- 轮播图区域，快捷键usp -->
		<swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular="true">
			<!-- 循环渲染轮播图的item项 -->
			<swiper-item v-for="(item, index) in swiperList" :key="index">
				<navigator class="swiper-item" :url="'/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id">
					<!-- 动态绑定图片的src属性 -->
					<image :src="item.image_src"></image>
				</navigator>
			</swiper-item>
		</swiper>
		<!-- 分类导航区域 -->
		<view class="nav-list">
			<view class="nav-item" v-for="(item, index) in navList" :key="index" @click="navClickHandler(item)"><image :src="item.image_src" class="nav-img"></image></view>
		</view>
		<!-- 楼层区域 -->
		<view class="floor-list">
			<!-- 楼层的item项 -->
			<view class="floor-item" v-for="(item, index) in floorList" :key="index">
				<!-- 楼层标题 -->
				<image :src="item.floor_title.image_src" :alt="item.floor_title.name" class="floor-title"></image>
				<!-- 楼层图片区域 -->
				<view class="floor-img-box">
					<!-- 左侧大图片的盒子 -->
					<navigator class="left-img-box" :url="item.product_list[0].url">
						<image :src="item.product_list[0].image_src" :style="{ width: item.product_list[0].image_width + 'rpx' }" mode="widthFix" :alt="item.product_list[0].name"></image>
					</navigator>
					<!-- 右侧小图片的盒子 -->
					<view class="right-img-box">
						<navigator class="right-img-item" v-for="(item2, index2) in item.product_list" :key="index2" v-if="index2 !== 0" :url="item2.url">
							<image :src="item2.image_src" :alt="item2.name" mode="widthFix" :style="{ width: item2.image_width + 'rpx' }"></image>
						</navigator>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
import badgeMix from '../../mixins/tabbar-badge.js'
	
export default {
	mixins: [badgeMix],
	data() {
		return {
			// 用于存储轮播图的数据列表
			swiperList: [],
			// 用于存储分类导航的数据列表
			navList: [],
			// 用于存储楼层的数据列表
			floorList: []
		};
	},
	onLoad() {
		// 调用获取轮播图的方法
		this.getSwiperList();
		// 调用获取分类导航的方法
		this.getNavList();
		// 调用获取楼层的方法
		this.getFloorList();
	},
	methods: {
		// 获取轮播图数据的方法
		async getSwiperList() {
			// 发起网络请求
			const { data: res } = await uni.$http.get('/api/public/v1/home/swiperdata');
			// 请求失败时
			if (res.meta.status !== 200) return uni.$showMsg();
			// 请求成功
			this.swiperList = res.message;
			console.log('轮播图', res.message);
		},
		// 获取分类导航的数据列表
		async getNavList() {
			// 发起网络请求
			const { data: res } = await uni.$http.get('/api/public/v1/home/catitems');
			// 请求失败时
			if (res.meta.status !== 200) return uni.$showMsg();
			// 请求成功时
			this.navList = res.message;
			console.log('分类导航', res.message);
		},
		// 获取楼层的列表数据
		async getFloorList() {
			// 发起网络请求
			const { data: res } = await uni.$http.get('/api/public/v1/home/floordata');
			// 请求失败时
			if (res.meta.status !== 200) return uni.$showMsg();
			// 请求成功时，通过双层forEach循环，处理URL地址
			res.message.forEach (floor => {
			  floor.product_list.forEach(prod => {
			    prod.url = '/subpkg/goods_list/goods_list?' + prod.navigator_url.split('?')[1]
			  })
			})
			this.floorList = res.message;
			console.log('楼层', res.message);
		},
		// 点击切换到分类页面
		navClickHandler(item) {
			if (item.name === '分类') {
				uni.switchTab({
					url: '/pages/cate/cate'
				});
			}
		},
		// 跳转到分包中的搜索页面
		gotoSearch() {
			uni.navigateTo({
				url: '/subpkg/search/search'
			})
		}
	}
};
</script>

<style lang="scss">
swiper {
	height: 330rpx;

	.swiper-item,
	image {
		width: 100%;
		height: 100%;
	}
}

.nav-list {
	display: flex;
	justify-content: space-around;
	margin: 15px 0;

	.nav-img {
		width: 128rpx;
		height: 140rpx;
	}
}

.floor-title {
	height: 60rpx;
	width: 100%;
	display: flex;
}
.right-img-box {
	display: flex;
	flex-wrap: wrap;
	justify-content: space-around;
}
.floor-img-box {
	display: flex;
	padding-left: 10rpx;
}

.search-box {
	// 设置定位效果为“吸顶”
	position: sticky;
	// 吸顶的“位置”
	top: 0;
	// 提高层级，防止被轮播图覆盖
	z-index: 999;
}
</style>
