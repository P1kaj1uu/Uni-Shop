<template>
	<view>
		<!-- 选择收货地址的盒子 -->
		<view class="address-choose-box" v-if="JSON.stringify(address) === '{}'">
			<button type="primary" size="mini" class="btnChooseAddress" @click="chooseAddress">请选择收货地址</button>
		</view>
		<!-- 渲染收货信息的盒子 -->
		<view class="address-info-box" v-else @click="chooseAddress">
			<!-- 第一行 -->
			<view class="row1">
				<!-- 第一行左边 -->
				<view class="row1-left">
					<view class="username">收货人：<text>{{address.userName}}</text></view>
				</view>
				<!-- 第一行右边 -->
				<view class="row1-right">
					<view class="phone">电话：<text>{{address.telNumber}}</text></view>
					<uni-icons type="arrowright" size="16"></uni-icons>
				</view>
			</view>
			<!-- 第二行 -->
			<view class="row2">
				<!-- 第二行左边 -->
				<view class="row2-left">收货地址:</view>
				<!-- 第二行右边 -->
				<view class="row2-right">{{addstr}}</view>
			</view>
		</view>
		<!-- 底部的边框线 -->
		<image src="../../static/cart_border@2x.png" class="address-border"></image>
	</view>
</template>

<script>
	import { mapState, mapMutations, mapGetters } from 'vuex'
	
	export default {
		name:"my-address",
		data() {
			return {
				
			}
		},
		methods: {
			// 把m_user模块中的updateAddress函数映射到当前组件
			...mapMutations('m_user', ['updateAddress']),
			// 点击选择收货地址
			async chooseAddress() {
				// 调用小程序提供的chooseAddress()方法，即可使用选择收货地址的功能
				const [err, succ] = await uni.chooseAddress().catch(err => err)
				// 用户成功的选择了收货地址
				if (err === null && succ.errMsg === 'chooseAddress:ok') {
					// 调用Store中提供的updateAddress方法，将address保存到Store里面
					this.updateAddress(succ)
				}
				// 用户没有授权
				if (err && (err.errMsg === 'chooseAddress:fail auth deny' || err.errMsg === 'chooseAddress:fail authorize no response')) {
					// 向用户重新发起授权申请
					this.reAuth()
				}
			},
			// 调用此方法，重新发起收货地址的授权
			async reAuth() {
			  // 提示用户对地址进行授权
			  const [err2, confirmResult] = await uni.showModal({
			    content: '检测到您没打开地址权限，是否去设置打开？',
			    confirmText: "确认",
			    cancelText: "取消"
			  })
			  // 如果弹框异常，则直接退出
			  if (err2) return
			  // 如果用户点击了 “取消” 按钮，则提示用户 “您取消了地址授权！”
			  if (confirmResult.cancel) return uni.$showMsg('您取消了地址授权！')
			  // 如果用户点击了 “确认” 按钮，则调用 uni.openSetting() 方法进入授权页面，让用户重新进行授权
			  if (confirmResult.confirm) return uni.openSetting({
			    // 授权结束，需要对授权的结果做进一步判断
			    success: (settingResult) => {
			      // 地址授权的值等于 true，提示用户 “授权成功”
			      if (settingResult.authSetting['scope.address']) return uni.$showMsg('授权成功！请选择地址')
			      // 地址授权的值等于 false，提示用户 “您取消了地址授权”
			      if (!settingResult.authSetting['scope.address']) return uni.$showMsg('您取消了地址授权！')
			    }
			  })
			}
		},
		computed: {
			// 把m_user模块中的address对象映射当前组件中使用，代替data中address对象
			...mapState('m_user', ['address']),
			...mapGetters('m_user', ['addstr']),
		}
	}
</script>

<style lang="scss">
.address-border {
	display: block;
	width: 100%;
	height: 5px;
}

.address-choose-box {
	display: flex;
	align-items: center;
	justify-content: center;
	height: 90px;
}

.address-info-box {
	display: flex;
	flex-direction: column;
	justify-content: center;
	padding: 0 5px;
  font-size: 12px;
  height: 90px;

  .row1 {
    display: flex;
    justify-content: space-between;

    .row1-right {
      display: flex;
      align-items: center;
			justify-content: space-between;

      .phone {
        margin-right: 5px;
      }
    }
  }

  .row2 {
    display: flex;
    align-items: center;
    margin-top: 10px;

    .row2-left {
      white-space: nowrap;
    }
  }
}
</style>
