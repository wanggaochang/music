<template>
  <view style="margin:0;padding: 0; background: url(../../static/back.png);" :style="{ height: swiperHeight + 'px' }">
	<uni-card
	style="margin: 0rpx 10rpx 0rpx 10rpx;"
	title="歌单列表" 
	:is-shadow="true"
	note="true"
	:style="{ background: 'linear-gradient(to right, rgba(71,161,230,'+x+'), rgba(71,161,230,'+(1-x)+'))' }"
	>
		<uni-collapse accordion="true" v-for="(item,index) in songsLists">
		    <uni-collapse-item 
			:title="item.listName"
			:style="{ background: 'linear-gradient(to right, rgba(152,251,152,'+x+'), rgba(152,251,152,'+(1-x)+'))' }"
			>
				<uni-list v-for="(song,songIndex) in item.listSongs">
					<uni-list-item
					:style="{ background: 'linear-gradient(to right, rgba(71,161,230,'+x+'), rgba(71,161,230,'+(1-x)+'))' }"
					>
						
						<view slot="header" class="slot-box viewContent">
							{{song.name}}
						</view>
						<template slot="footer">
							<image class="slot-image" src="../../static/delete.png" style="height: 40rpx;width: 40rpx;" @click="deleteSong(index,songIndex)"></image>
						</template>
					</uni-list-item>
				</uni-list>
		    </uni-collapse-item>
		</uni-collapse>
		<template v-slot:footer>
		    <view class="footer-box">
		        <image class="slot-image" src="../../static/plus.png" style="height: 40rpx;width: 40rpx;" @click="addList"></image>
				<view style="width: 40rpx;display: inline-block;"></view>
				<image class="slot-image" src="../../static/delete.png" style="height: 40rpx;width: 40rpx;" @click="deleteList"></image>
		    </view>
		</template>
	</uni-card>
	<!-- 弹出层 -->
	<uni-popup ref="addList" type="top">
		<view style="width: 750rpx;background-color: #007AFF;" >
			<uni-search-bar :radius="30" :value="addListName" @input="onKeyInput" cancelButton="none"></uni-search-bar>
			<button @click="saveList">确定</button>
		</view>
	</uni-popup>
	<uni-popup ref="deleteList" type="bottom">
		<view style="width: 750rpx">
			<uni-link v-for="(item,index) in songsLists">
				<uni-list-item
				:style="{ background: 'linear-gradient(to right, rgba(71,161,230,'+x+'), rgba(71,161,230,'+(1-x)+'))' }"
				>
					<view slot="header" class="slot-box">
						{{item.listName}}
					</view>
					<template slot="footer">
						<image class="slot-image" src="../../static/delete.png" style="height: 40rpx;width: 40rpx;" @click="deleteThisList(index)"></image>
					</template>
				</uni-list-item>
			</uni-link>
		</view>
	</uni-popup>
	<uni-popup ref="tips" type="center">
		<view style="width: 400rpx;">
			还没有歌单，点击'+'新建一个吧!
		</view>
	</uni-popup>
  </view>
</template>

<script>
	export default {
		data() {
			return {
				songsLists:[],
				addListName:'',
				x: 0,
				flag: true,
				swiperHeight:''
			}
		},
		onLoad() {
			setInterval(() => {
				var that = this;
				uni.getStorage({
				    key: 'songsListKey',
				    success: function (res) {
				        that.songsLists = res.data;	
				    }
				})
			},1000)
			uni.getSystemInfo({
			        success:  (res) => {     // 需要使用箭头函数，swiper 高度才能设置成功
			            // 获取swiperHeight可以获取的高度，窗口高度减去导航栏高度 - uni.upx2px(25)
			            this.swiperHeight = res.windowHeight
			            console.log(this.swiperHeight)
			        }
			})
		},
		onShow() {
			if(this.songsLists.length===0){
				this.$refs.tips.open()
			}
		},
		onReady() {
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
			},30);
		},
		methods: {
			// 缓存歌单数据
			setData(){
				var that = this;
				uni.setStorage({
				    key: 'songsListKey',
				    data: that.songsLists
				});
			},
			// 实时获取输入框的值
			onKeyInput: function(event) {
			    this.addListName = event;
			},
			addList(){
				this.$refs.addList.open();
			},
			saveList(){
				var songsList = new Object();
				songsList.listName=this.addListName;
				songsList.listSongs = [];
				this.songsLists.push(songsList);
				this.addListName='';
				this.$refs.addList.close();
				this.setData();
			},
			deleteList(){
				this.$refs.deleteList.open();
			},
			deleteThisList(index){
				let temp = [];
				for(let i=0;i<this.songsLists.length;i++){
					if(i != index){
						temp.push(this.songsLists[i]);
					}
				}
				this.songsLists = temp;
				this.setData();
			},
			deleteSong(index,songIndex){
				let temp = [];
				for(let i=0;i<this.songsLists[index].listSongs.length;i++){
					if(i != songIndex){
						temp.push(this.songsLists[index].listSongs[i]);
					}
				}
				this.songsLists[index].listSongs = temp;
				this.setData();
			}
		}
	}
</script>

<style>
	.viewContent{
			display: -webkit-box;
			-webkit-box-orient:vertical;
			text-overflow: ellipsis;
			overflow: hidden;
			-webkit-line-clamp: 1;
			width: 450rpx;
		}
</style>
