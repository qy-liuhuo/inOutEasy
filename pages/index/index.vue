<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>
		<uni-popup ref="popup" type="message">
		    <uni-popup-message type="warn" message="无法获取openId" :duration="5000"></uni-popup-message>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '抗击疫情，我们在一起！'
			}
		},
		onLoad() {
			uni.login({
			  provider: 'weixin',
			  success: function (loginRes) { 
				//console.log(loginRes.code);
				//通过code请求用户标识
				uni.request({
				    url: 'https://accessmanage.qylh.xyz/index.php/client/index/login',
				    data: {
				        code: loginRes.code,
				    },
					method:"POST",
				    success: (res) => {
						uni.setStorageSync('openId',res.data.openid);
						uni.setStorageSync('name',res.data.name);
						console.log(uni.getStorageSync('name'));

						uni.switchTab({
							url: '/pages/registration/registration'
						});
						
				    },
					error: ()=>{
						this.$refs.popup.open();
						console.log('无法获取OPENID');
					}
				}); 
			  }
			});
		},
		methods: {

		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
