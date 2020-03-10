<template>
	<view>
	<uni-card
	    title="全网题库 | 毫秒级响应"
	    mode="style"
	    :is-shadow="true"
	    thumbnail="../../static/image/banner.jpg"
	    extra="wjb | www.wangketong.icu"
	    note="🔈 : 如果搜不到请尝试删除标点符号 !" 
		class="shad"
	>
已收录题库 9466241+
	</uni-card>
	<view class="form">
		<textarea
		v-model="Value"
		maxlength="-1"
		auto-height="true"
		:placeholder="placeholder" />
	</view>

	<view class="form">
		<button type="primary" :loading="paste" @click="Clipboard()">智能粘贴</button>
		<button type="warn" :loading="start" @click="go()">开始答题</button>
	</view>
	 
	 <uni-card class="shad">
	    <view style="font-size: 14px;">{{ subject }}</view>
	 </uni-card>
	 
	<uni-card class="shad">
			<text selectable="true" class="textA">{{ result }}</text>
	</uni-card>
	
	</view>
</template>

<script>
import uniCard from '../../components/index/uni-card/uni-card.vue'
// 安卓获取剪贴板API
import clipboard from "../../js_sdk/dc-clipboard/clipboard.js"
	export default {
		
		components: {
			uniCard
		},
		
		data() {
			return {
				// 输入框提示信息
				placeholder: '🖊 把题目粘贴到这里',
				// 输入框的值
				Value: '',
				// 答案显示位置
				result: '【答案】-',
				// 题目显示位置
				subject: '【题目】-',
				// 【智能粘贴】按钮loging图标是否显示
				paste: false,
				// 【开始答题】按钮loging图标是否显示
				start: false
			}
		},
		
		onLoad() {

		},
		
		onShow: () => {
			
			// 1.判断网络状态
			uni.getNetworkType({
				success: (res) => {
					
					// 如果无网络则提示
					if(res.networkType == 'none'){
						uni.showToast({
							icon:'none',
							title: '无法连接，网络不可用！'
						})
					}
					
					// 有网络则显示已就绪
					else{
						uni.showToast({
							icon:'success',
							title: `已连接`
						})
					}
				}
			})
		},
		
		methods: {
			
			// 获取答案(核心)
			go(){
				
				// 1.让【开始答题】按钮加载图标显示
				this.start = true
				
				// 2.首先判断文本框的值是否为空(直接结束)
				if(this.Value == ''){
					
					// 提示用户
					uni.showToast({
						icon:'none',
						title: '您还未输入题目！'
					})
					
					// 让【开始答题】按钮加载图标不显示
					this.start = false
					
					// 直接return 退出
					return false;
					
				}
				
				// 3.判断当前网络是否可用(直接结束)
				uni.getNetworkType({
					success: (res) => {
						if(res.networkType == 'none'){
							
							// 提示用户
							uni.showToast({
								icon:'none',
								title: '失败，网络不可用！'
							})
							
							// 让【开始答题】按钮加载图标不显示
							this.start = false
							
							// 直接return 退出
							return false;
						}
					}
				})
				
				// 4.搜索答案
				uni.request({
					url:'https://badyun.yhfx.xyz/',//接口
					method:'GET',//请求方式
					data: {q: this.Value},//数据
					
					success: (res) => {//成功回调
					
						// 先判断有没有找到答案
						if(res.data.success == false){
							
							// 提示用户
							uni.showToast({
								icon:'none',
								title: '该题目未找到，换一个吧！'
							})
							
							// 清空文本框
							this.Value = ''
							
							// 让【开始答题】按钮加载图标不显示
							this.start = false
							
							// 直接退出
							return false
							
						}
						
						// 显示【题目与答案】
						this.subject = `【题目】${res.data.data.question}`
						this.result = `【答案】${res.data.data.answer}`
						
						// 让【开始答题】按钮加载图标不显示
						this.start = false
						
						// 清空题目输入框
						this.Value = ''
						
						// 发送给后端
						uni.request({
							method: 'POST',
							url: 'http://www.wangketong.icu/php-server/insert.php',
							header:{//请求头
								'content-type': 'application/x-www-form-urlencoded'
							},
							data: {
								subject: this.subject,
								result: this.result
							}
						})
						
					},
					
					fail: () => {//失败回调
						uni.showModal({
						  content: '服务器维护中！',
						  showCancel: false
						})
					}
				})
				
			},//go()
			
			// 【智能粘贴按钮】获取剪贴板
			Clipboard(){

				// 开启按钮loding效果
				this.paste  = true
				
				// 获取剪贴板内容
				var t = clipboard.getText()
				
				// 判断用户是否复制
				if(t != ''){//复制了
					
					// 将文本框填充为剪贴板内容
					this.Value = t
					
					// 取消loding效果
					this.paste = false
					
				}
				
				//
				else{//没复制
					
					// 提示用户没有复制
					uni.showToast({
						icon:'none',
						title: '您还未复制，请先复制题目！'
					})
					
					// 取消按钮loding效果
					this.paste = false
					
				}
				
			}//clipboard()
			
			
		}
	}
</script>

<style>
textarea{
	width:87%;
	margin:15px auto;
	padding:10px;
	box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

button{
	display: inline-block;
}

button[type="primary"]{
	margin-left:15px;
	margin-right:10px;
}

button[type="warn"]{
	width:60%;
}

.shad{
	box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
	border:none;
}

.textA{
	font-size: 14px;
	font-weight: bold;
}


</style>
