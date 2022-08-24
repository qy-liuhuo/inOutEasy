<template>
	<view>
	<view id="recordForm">
		<u--form labelPosition="left" :model="record" ref="form1" :rules="rules">
			<u-button type="warning"  :plain="true" text="跳转到健康码" @click="JKMredirect"></u-button>
			<u-form-item
					label="姓名"
					prop="name"
					borderBottom
					ref="item1"	
			>
				<u--input
						v-model="record.name" v-bind:disabled="!finished"
				></u--input>
			</u-form-item>
			
			<u-form-item
					label="方向"
					prop="direction"
					borderBottom
					@click="showDirection = true; hideKeyboard()"
					ref="item1"
			>
				<u--input
						v-model="record.direction"
						disabled
						disabledColor="#ffffff"
						placeholder="请选择进出方向"
						border="none"
				></u--input>
				<u-icon
						slot="right"
						name="arrow-right"
				></u-icon>
			</u-form-item>
			<u-form-item  label="体温" prop="temperature" ref="item1">
				<u-input v-model="record.temperature" placeholder="请进行测温" disabled>
					<template slot="suffix">
						<u-code ref="uCode" @change="codeChange" seconds="20" start-text="测温" change-text="X秒后重新测温" end-text="重新测温"></u-code>
						<u-button @tap="getTemperature" :text="tips" type="success"size="mini"></u-button>
					</template>
				</u-input>
			</u-form-item>
		</u--form>
		<br/>
		<u-action-sheet
				:show="showDirection"
				:actions="actions"
				title="进出方向"
				@close="showDirection = false"
				@select="directionSelect"
		>
		</u-action-sheet>
	</view>
	<view><u-button id="submitButton" type="primary" text="提交" @click="submit"></u-button></view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			tips: '',
			showDirection: false,
			finished:(uni.getStorageSync('name')==null)?1:0, 
			record: {
					name: uni.getStorageSync('name'),
					openId: uni.getStorageSync('openId'),
					direction:'进',
					temperature:null,
			},
			actions: [{
				name: '进',
				},
				{
					name: '出',
				},
			],
			rules: {
				'name': {
					type: 'string',
					required: true,
					message: '请填写姓名',
					trigger: ['blur', 'change']
				},
				'direction': {
					type: 'string',
					required: true,
					message: '请填写进出方向',
					trigger: ['blur', 'change']
				}
			},
			radio: '',
			switchVal: false
		};
	},
	onReady() {
		// 如果需要兼容微信小程序，并且校验规则中含有方法等，只能通过setRules方法设置规则
		this.$refs.form1.setRules(this.rules)
	},
	methods: {
		directionSelect(e) {
			this.record.direction = e.name
			this.$refs.form1.validateField('record.direction')
		},
		getTemperature()
		{
			if (this.$refs.uCode.canGetCode) {
				  // 模拟向后端请求验证码
			  uni.showLoading({
				title: '正在请求测温'
			  })
				uni.request({
					url: 'https://accessmanage.qylh.xyz/index.php/client/index/getTemperature',
					method:'GET',
					success: (res) => {
						uni.hideLoading();
						let code=res.data.data.requestCode;
						let temperature=res.data.data.temperature;
						if(code==200)
						{
							this.record.temperature=temperature;
							uni.showModal({
								content: '测温成功',
								showCancel: false
							});	
						}
						else
						{
							uni.showModal({
								content: '测温超时',
								showCancel: false
							});	
						}
					},
					fail:(err)=>{
						uni.showModal({
							content: '测温请求失败，请检查网络',
							showCancel: false
						});	
					}
				});
				} else {
				  uni.$u.toast('倒计时结束后重新请求');
			}
		},
	    codeChange(text) {
			  this.tips = text;
		},
		JKMredirect()
		{
			let openID=null;
			uni.request({
			    url: 'https://accessmanage.qylh.xyz/index.php/client/index/getJkm',
				method:'GET',
			    success: (res) => {
					openID=res.data.openid;
					uni.navigateToMiniProgram({
					  appId: openID,
					  path: '', 
					  extraData: {
					    'data1': 'test'
					  },
					  success(res) {
					    // 打开成功  
					  },
					});
			    },
				fail:(err)=>{
					uni.showModal({
						content: '自动打开失败请重新手动打开健康码',
						showCancel: false
					});	
				}
			});
		},
		submitRecord()
		{
			var formdata=this.record;
			uni.request({
			    url: 'https://accessmanage.qylh.xyz/index.php/client/index/checkPass',
			    data: formdata,
				method:'POST',
			    success: (res) => {
					if(res.data.result=="allow")
					{
						uni.$u.toast('请通行');
					}
					else
					{
						uni.$u.toast(res.data.msg);
					}
			    },
				fail:(err)=>{
					console.log(err);
				}
			});
		},
		submit()
		{
			this.$refs.form1.validate().then(res => {
				if(this.record.temperature==null)
				{
					uni.$u.toast('请进行测温')
				}
				else
				{
					this.submitRecord();
				}
			}).catch(errors => {
				console.log(errors);
				uni.$u.toast('请填写完整')
			})
		},
		
	},
};
</script>
<style>
	#recordForm{
		margin-top:10rpx;
		margin-bottom:20rpx;
		padding-left:20px;
		padding-right:20px;
	}
	#submitButton
	{
		margin-top: 20rpx;
	}
</style>