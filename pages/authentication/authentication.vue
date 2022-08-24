<template>
    <view style="padding: 40rpx;">
			<p>注：该信息仅用于与数据库中数据进行匹配以完成用户认证，您填写的信息将不会被存储，临时测试请勿填写真实信息</p>
			<hr/>
			<br/>
            <form @submit="formSubmit" @reset="formReset">
                <view class="uni-form-item uni-column">
                    <view class="title">用户名</view>
                    <view>
                        <input name="name" v-model="name" v-bind:disabled="!finished" type="text" placeholder="请输入用户名" />
                    </view>
                </view>
				<view v-if="finished" class="uni-form-item uni-column">
				    <view class="title">认证ID</view>
				    <view>
				        <input name="idCode" type="text" placeholder="请输入ID" />
				    </view>
				</view>
                
                <view class="uni-btn-v" style="margin-top:20px;">
                    <button v-if="finished" type="primary" form-type="submit">提交</button>
                    <button v-else type="default" disabled="disable" style="color: #18BC37" >已完成认证</button>
                </view>
            </form>
			<uni-popup ref="message" type="message" :backgroundColor="popupColor">
				<uni-popup-message :type="msgType" :message="messageText" :duration="2000"></uni-popup-message>
			</uni-popup>
    </view>
</template>
<script>
    export default {
        data() {
            return {
				finished:(uni.getStorageSync('name')==null)?1:0, 
				name:uni.getStorageSync('name'),
				messageText:'',
				popupColor:'',
            }
        },
		created	() {
		 console.log(uni.getStorageSync('name'));
		},
        methods: {
            formSubmit: function(e) {
                console.log(JSON.stringify(e.detail.value))
                var formdata = e.detail.value
				formdata.openid=uni.getStorageSync('openId');
				//请求认证
                uni.request({
                    url: 'https://accessmanage.qylh.xyz/index.php/client/index/authentication',
                    data: formdata,
					method:'POST',
                    success: (res) => {
						res=res.data;
						if(res.code==1)
						{
							this.$data.messageText='认证成功';
							this.$data.popupColor='#2ecc71';
							uni.setStorageSync('name',res.info.name);
							uni.switchTab({
							    url: '/pages/registration/registration'
							});
						}
						else
						{
							this.$data.messageText='认证失败'
							this.$data.popupColor='#ff6348'
						}
						this.$refs.message.open('message');
                    }
                });
            },
            formReset: function(e) {
                console.log('清空数据')
            }
        }
    }
</script>

<style>
    .uni-form-item .title {
        padding: 20rpx 0;
    }
</style>