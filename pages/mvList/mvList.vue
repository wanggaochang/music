<template>
	<view style="margin:0;padding: 0; background: url(../../static/back.png);" :style="{ height: swiperHeight + 'px' }">
		<!-- 导航栏 -->
		<view style="margin: 0;" :style="{ height: swiperHeight/8 + 'px' }">
			<view :style="{ background: 'linear-gradient(to right, rgba(71,161,230,'+x+'), rgba(71,161,230,'+(1-x)+'))' }">
				<view style="display: flex;flex-direction: row;padding: 10rpx;">
					<view style="flex: 3;">地区：</view>
					<view style="flex: 3;" @click="changeArea('全部')">全部</view>
					<view style="flex: 3;" @click="changeArea('内地')">内地</view>
					<view style="flex: 3;" @click="changeArea('港台')">港台</view>
					<view style="flex: 3;" @click="changeArea('欧美')">欧美</view>
					<view style="flex: 3;" @click="changeArea('日本')">日本</view>
					<view style="flex: 3;" @click="changeArea('韩国')">韩国</view>
					<view style="flex: 3;"></view>
				</view>
				<view style="display: flex;flex-direction: row;padding: 10rpx;">
					<view style="flex: 3;">类型：</view>
					<view style="flex: 3;" @click="changeType('全部')">全部</view>
					<view style="flex: 4;" @click="changeType('官方版')">官方版</view>
					<view style="flex: 3;" @click="changeType('原声')">原声</view>
					<view style="flex: 4;" @click="changeType('现场版')">现场版</view>
					<view style="flex: 5;" @click="changeType('网易出品')">网易出品</view>
					<view style="flex: 2;"></view>
				</view>
				<view style="display: flex;flex-direction: row;padding: 10rpx;">
					<view style="flex: 3;">排序：</view>
					<view style="flex: 5;" @click="changeOrder('上升最快')">上升最快</view>
					<view style="flex: 3;" @click="changeOrder('最热')">最热</view>
					<view style="flex: 3;" @click="changeOrder('最新')">最新</view>
					<view style="flex: 10;"></view>
				</view>
			</view>
		</view>
		<!-- mv列表 -->
		 最近筛选： {{area}}  {{type}}  {{order}}
		<scroll-view scroll-y @scrolltolower="getMoreMv" style="padding: 10rpx 20rpx 0rpx 20rpx;" :style="{ height: swiperHeight/6*5 + 'px' }">
			<view v-for="(item,index) in mvList">
				<view style="padding: 20rpx 0rpx 20rpx 0rpx ;">
					<image style="width: 710rpx;" :src="item.cover" @click="playMv(index)"></image>
					<view>
						{{item.name==='null'?'':item.name}}
						<view style="width: 20rpx; display: inline-block;"></view>
						{{item.briefDesc==='null'?'':item.briefDesc}}
						<view style="width: 20rpx; display: inline-block;"></view>
						{{item.artistName==='null'?'':item.artistName}}
					</view>
				</view>
			</view>
		</scroll-view>
		<!-- mv播放弹出框 -->
		<uni-popup ref="playMv" type="center">
				<video
					:src="mvUrl"
					autoplay
					controls
					style="width: 750rpx;height: 562.5;"
				></video>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				swiperHeight:'',
				area:'全部',
				order:'上升最快',
				type:'全部',
				mvList:[],
				x: 0,
				flag: true,
				mvUrl:'',
				limit: 10,
				offset: 0
			}
		},
		onPullDownRefresh(){
			this.searchMv();
			setTimeout(function () {
			            uni.stopPullDownRefresh();
			        }, 1000);
		},
		onLoad() {
			uni.getSystemInfo({
			        success:  (res) => {     // 需要使用箭头函数，swiper 高度才能设置成功
			            // 获取swiperHeight可以获取的高度，窗口高度减去导航栏高度 - uni.upx2px(25)
			            this.swiperHeight = res.windowHeight
			            console.log(this.swiperHeight)
			        }
			}),
			setInterval(() => {
				//开始flag为true x=0
				if(this.flag && this.x<1){
					this.x=this.x+0.01;
				}else if(this.x-1>0 && this.x-1<0.01){
					this.flag=false;
					this.x=this.x-0.01;
				}else if(!this.flag && this.x>0.01){
					this.x=this.x-0.01;
				}else if(this.x>0&&this.x<0.01){
					this.flag=true;
					this.x=this.x+0.01;
				}
			},30),
			this.searchMv()
		},
		methods: {
			changeArea(prmar){
				this.area = prmar;
			},
			changeOrder(prmar){
				this.order = prmar;
			},
			changeType(prmar){
				this.type = prmar;
			},
			searchMv(){
				uni.request({
					url:'https://autumnfish.cn/mv/all',
					data:{
						area: this.area,
						type: this.type,
						order: this.order,
						limit: this.limit,
						offset: this.offset
					},
					header:{
						'custom-header': 'searchMv'
					},
					success:(res) => {
							this.mvList=res.data.data;
					}
				})
			},
			playMv(index){
				uni.request({
					url:'https://autumnfish.cn/mv/url',
					data:{
						id: this.mvList[index].id
					},
					header:{
						'custom-header': 'playMvByMvid'
					},
					success: (res) => {
						if(res.data.code===200){
							this.mvUrl=res.data.data.url;
							this.$refs.playMv.open();
						}
					}
				})
			},
			getMoreMv(){
				this.offset+=10;
				this.limit+=10;
				uni.request({
					url:'https://autumnfish.cn/mv/all',
					data:{
						area: this.area,
						type: this.type,
						order: this.order,
						limit: this.limit,
						offset: this.offset
					},
					header:{
						'custom-header': 'searchMv'
					},
					success:(res) => {
						for(let i=0;i<res.data.data.length;i++){
							this.mvList.push(res.data.data[i]);
						}
					}
				})
			}
		},
		watch:{
			area(area){
				this.searchMv();
			},
			order(order){
				this.searchMv();
			},
			type(type){
				this.searchMv();
			}
		}
	}
</script>

<style>

</style>
