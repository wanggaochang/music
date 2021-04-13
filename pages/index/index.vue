<template>
	<view style="padding: 0; background: url(../../static/back.png);background-repeat: repeat-y;" :style="{ height: swiperHeight + 'px' }">
	<!-- 顶部搜索框及搜索按钮 -->
		<view :style="{ height:swiperHeight/12 + 'px' }">
			<view style="padding: 8rpx 30rpx 30rpx 30rpx; flex-direction: row;display: flex;">
				<!-- 搜索框 -->
				<view style="flex: 8;">
					<uni-search-bar :radius="60" :value="keyWord" @input="onKeyInput" cancelButton="none"></uni-search-bar>
				</view>
				<!-- 搜索按钮 -->
				<view  style="padding-top: 20rpx;" >
					<uni-icons type="search" size="30" style="color: #007AFF;" @click="searchMusic"></uni-icons>
				</view>
			</view>
		</view>
		
		<!-- 歌单和收藏按钮 -->
		<scroll-view scroll-y = "true"  :style="{ height: swiperHeight/8*5 + 'px' }" style="margin: 0;">
			<uni-notice-bar style="margin: 5rpx 0rpx 0rpx 0rpx;padding: 0rpx;" showClose = "true" scrollable="true" single="true" text="下拉可唤出歌单列表,可选取! QQ2208165599!!！"></uni-notice-bar>
			<uni-card 
				style="margin: 5rpx 10rpx 0rpx 10rpx;"
				:title="listName" 
				mode="style" 
				:is-shadow="true"
				:thumbnail="picUrl[picUrlIndex]" 
				>
				<view>
					<uni-list v-for="(item,index) in musicList">
					    <uni-list-item
						:style="{ background: 'linear-gradient(to right, rgba(71,161,230,'+x+'), rgba(71,161,230,'+(1-x)+'))' }"
							>
							 <!-- 自定义 header -->
							<view slot="header" class="slot-box">
								{{item.name}}
							</view>
							<!-- 自定义 body -->
							 
							<!-- 自定义 footer-->
							<template slot="footer">
								<image class="slot-image" v-show="!item.mvid==0" src="../../static/mv.png" style="height: 40rpx;width: 40rpx;" @click="playMv(index)"></image>
								<view style="width: 20rpx;"></view>
								<image class="slot-image" v-show="item.isAdd===0" src="../../static/add.png" style="height: 40rpx;width: 40rpx;" @click="addMusic(index)"></image>
								<image class="slot-image" v-show="item.isAdd===1" src="../../static/addTrue.png" style="height: 40rpx;width: 40rpx;" @click="addMusic(index)"></image>
								<view style="width: 20rpx;"></view>
							    <image class="slot-image" v-show="item.rtype===0" src="../../static/play.png" style="height: 40rpx;width: 40rpx;" @click="playMusic(item.id,index)"></image>
							    <image class="slot-image" v-show="item.rtype===1" src="../../static/pause.png" style="height: 40rpx;width: 40rpx;" @click="playMusic(item.id,index)"></image>
							</template>
						</uni-list-item>
					</uni-list>
				</view>
			</uni-card>
		</scroll-view>
		<!-- 底部音乐播放进度条 -->
		<view :style="{ height: swiperHeight/12 + 'px' }" style="margin: 0rpx;">
			 <uni-card
			 :title="now==-1 ? '请选择歌曲点击播放' : musicList[now].name"
			style="margin: 5rpx 10rpx 0rpx 10rpx;"
			:style="{ background: 'linear-gradient(to right, rgba(71,161,230,'+x+'), rgba(71,161,230,'+(1-x)+'))'}"
			>
				<imt-audio autoplay continue :src="now==-1?'':playUrlList[now]"
				 @prev="now = now === 0?playUrlList.length-1:now-1" 
				@next="now = now === playUrlList.length-1?0:now+1"></imt-audio>
			 </uni-card>
		</view>
		<!-- 弹出层 -->
		<uni-popup ref="songsLists" type="bottom">
			<view style="width: 750rpx">
				<uni-link title="选择要添加的歌单" v-for="(item,index) in songsLists">
					<uni-list-item 
					clickable 
					@click="addMusicToList(index)"
					:style="{ background: 'linear-gradient(to right, rgba(71,161,230,'+x+'), rgba(71,161,230,'+(1-x)+'))' }"
					>
						<view slot="header" class="slot-box">
							{{item.listName}}
						</view>
					</uni-list-item>
				</uni-link>
			</view>
		</uni-popup>
		
		<!-- 歌单播放列表弹出框 -->
		<uni-popup ref="playSongsLists" type="top">
			<view style="width: 750rpx">
				<uni-link title="所有歌单" v-for="(item,index) in songsLists">
					<uni-list-item 
					clickable 
					@click="playSongsList(index)"
					:style="{ background: 'linear-gradient(to right, rgba(71,161,230,'+x+'), rgba(71,161,230,'+(1-x)+'))' }"
					>
						<view slot="header" class="slot-box">
							{{item.listName}}
						</view>
					</uni-list-item>
				</uni-link>
			</view>
		</uni-popup>
		
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
				title: 'Hello',
				myaudio:uni.createInnerAudioContext({}),
				keyWord: '',
				musicList: [],
				playList: [],
				playUrlList: [],
				swiperHeight: '',
				picUrl:[
					'../../static/shaosiming.png',
					'../../static/shaosiming1.png',
					'../../static/shaosiming2.png',
					'../../static/shaosiming3.png',
					'../../static/shaosiming4.png',
					'../../static/shaosiming5.png'
					],
				picUrlIndex: '0',
				x: 0,
				flag: true,
				now: '-1',
				songsLists: '',
				addSong: '',
				listName: '歌曲列表',
				mvUrl:''
			}
		},
		components: {
			
		},
		// 下拉唤出歌单列表
		onPullDownRefresh(){
			this.$refs.playSongsLists.open();
			setTimeout(function () {
			            uni.stopPullDownRefresh();
			        }, 1000);
		},
		onLoad() {
			// 读取缓存
			setInterval(() => {
				var that = this;
				uni.getStorage({
				    key: 'songsListKey',
				    success: function (res) {
				        that.songsLists = res.data;	
				    }
				})
			},1000);
			
			// 获取屏幕高度
			this.swiperHeight = uni.getSystemInfoSync().windowHeight,
			// 初始化数据
				this.musicList=[
					
				{
				    "id": 1490362890,
				    "name": "笑纳",
				    "artists": [
				        {
				            "id": 37280642,
				            "name": "子伟",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p2.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        }
				    ],
				    "album": {
				        "id": 97412271,
				        "name": "笑纳 (男版)",
				        "artist": {
				            "id": 0,
				            "name": "",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p2.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        },
				        "publishTime": 1603728000000,
				        "size": 3,
				        "copyrightId": 1416960,
				        "status": 0,
				        "picId": 109951165418713840,
				        "mark": 0
				    },
				    "duration": 202710,
				    "copyrightId": 1416960,
				    "status": 0,
				    "alias": [],
				    "rtype": 0,
				    "ftype": 0,
				    "mvid": 0,
				    "fee": 8,
				    "rUrl": null,
				    "mark": 8192
				},
				{
				    "id": 516076896,
				    "name": "纸短情长",
				    "artists": [
				        {
				            "id": 1021160,
				            "name": "烟把儿",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p1.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        }
				    ],
				    "album": {
				        "id": 36686617,
				        "name": "纸短情长",
				        "artist": {
				            "id": 0,
				            "name": "",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p1.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        },
				        "publishTime": 1509206400000,
				        "size": 4,
				        "copyrightId": 1416618,
				        "status": 0,
				        "picId": 109951165772646320,
				        "mark": 0
				    },
				    "duration": 300173,
				    "copyrightId": 1416618,
				    "status": 0,
				    "alias": [],
				    "rtype": 0,
				    "ftype": 0,
				    "mvid": 5810734,
				    "fee": 8,
				    "rUrl": null,
				    "mark": 8256
				},
				{
				    "id": 25706282,
				    "name": "夜空中最亮的星",
				    "artists": [
				        {
				            "id": 12977,
				            "name": "逃跑计划",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p2.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        }
				    ],
				    "album": {
				        "id": 2285010,
				        "name": "世界",
				        "artist": {
				            "id": 0,
				            "name": "",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p2.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        },
				        "publishTime": 1325260800000,
				        "size": 10,
				        "copyrightId": 22036,
				        "status": 1,
				        "picId": 109951165543196750,
				        "mark": 0
				    },
				    "duration": 252000,
				    "copyrightId": 22036,
				    "status": 0,
				    "alias": [],
				    "rtype": 0,
				    "ftype": 0,
				    "mvid": 382555,
				    "fee": 8,
				    "rUrl": null,
				    "mark": 8192
				},
				{
				    "id": 4876053,
				    "name": "绝世",
				    "artists": [
				        {
				            "id": 6500,
				            "name": "张克帆",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p2.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        }
				    ],
				    "album": {
				        "id": 489978,
				        "name": "水月洞天 电视原声带",
				        "artist": {
				            "id": 0,
				            "name": "",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p2.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        },
				        "publishTime": 1091116800000,
				        "size": 28,
				        "copyrightId": 684010,
				        "status": 1,
				        "picId": 89060441864804,
				        "mark": 0
				    },
				    "duration": 176909,
				    "copyrightId": 684010,
				    "status": 0,
				    "alias": [
				        "电视剧《水月洞天》主题曲"
				    ],
				    "rtype": 0,
				    "ftype": 0,
				    "mvid": 0,
				    "fee": 8,
				    "rUrl": null,
				    "mark": 1125899906851328
				},
				{
				    "id": 526464293,
				    "name": "空空如也 ",
				    "artists": [
				        {
				            "id": 9255,
				            "name": "任然",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p1.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        }
				    ],
				    "album": {
				        "id": 37087074,
				        "name": "空空如也",
				        "artist": {
				            "id": 0,
				            "name": "",
				            "picUrl": null,
				            "alias": [],
				            "albumSize": 0,
				            "picId": 0,
				            "img1v1Url": "https://p1.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				            "img1v1": 0,
				            "trans": null
				        },
				        "publishTime": 1514390400007,
				        "size": 2,
				        "copyrightId": 30002,
				        "status": 0,
				        "picId": 109951163094476380,
				        "mark": 0
				    },
				    "duration": 213846,
				    "copyrightId": 1416392,
				    "status": 0,
				    "alias": [],
				    "rtype": 0,
				    "ftype": 0,
				    "mvid": 10931476,
				    "fee": 8,
				    "rUrl": null,
				    "mark": 0
				},
				{
				                "id": 431554147,
				                "name": "逍遥叹－徐薇（Cover 胡歌）",
				                "artists": [
				                    {
				                        "id": 978026,
				                        "name": "徐薇",
				                        "picUrl": null,
				                        "alias": [],
				                        "albumSize": 0,
				                        "picId": 0,
				                        "img1v1Url": "https://p1.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				                        "img1v1": 0,
				                        "trans": null
				                    }
				                ],
				                "album": {
				                    "id": 34890056,
				                    "name": "徐薇翻唱合集",
				                    "artist": {
				                        "id": 0,
				                        "name": "",
				                        "picUrl": null,
				                        "alias": [],
				                        "albumSize": 0,
				                        "picId": 0,
				                        "img1v1Url": "https://p1.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
				                        "img1v1": 0,
				                        "trans": null
				                    },
				                    "publishTime": 1405267200000,
				                    "size": 59,
				                    "copyrightId": 0,
				                    "status": 1,
				                    "picId": 109951162860961680,
				                    "mark": 0
				                },
				                "duration": 313304,
				                "copyrightId": 0,
				                "status": 0,
				                "alias": [],
				                "rtype": 0,
				                "ftype": 0,
				                "mvid": 0,
				                "fee": 0,
				                "rUrl": null,
				                "mark": 0
				            },
							 {
							        "id": 452986458,
							        "name": "红昭愿",
							        "artists": [
							            {
							                "id": 12174521,
							                "name": "音阙诗听",
							                "picUrl": null,
							                "alias": [],
							                "albumSize": 0,
							                "picId": 0,
							                "img1v1Url": "https://p2.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
							                "img1v1": 0,
							                "trans": null
							            }
							        ],
							        "album": {
							            "id": 35114938,
							            "name": "红昭愿",
							            "artist": {
							                "id": 0,
							                "name": "",
							                "picUrl": null,
							                "alias": [],
							                "albumSize": 0,
							                "picId": 0,
							                "img1v1Url": "https://p2.music.126.net/6y-UleORITEDbvrOLV0Q8A==/5639395138885805.jpg",
							                "img1v1": 0,
							                "trans": null
							            },
							            "publishTime": 1484064000007,
							            "size": 2,
							            "copyrightId": 30002,
							            "status": 0,
							            "picId": 109951162951242160,
							            "mark": 0
							        },
							        "duration": 173217,
							        "copyrightId": 1416678,
							        "status": 0,
							        "alias": [],
							        "rtype": 0,
							        "ftype": 0,
							        "mvid": 0,
							        "fee": 8,
							        "rUrl": null,
							        "mark": 8256
							    }
				]
				
		},
		onReady() {
			setInterval(() => {
				if(this.picUrlIndex<6){
					this.picUrlIndex++;
				}else{
					this.picUrlIndex=0;
				}
			},2000);
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
			{
				for(let i=0;i<this.musicList.length;i++){
					uni.request({
					    url: 'https://autumnfish.cn/song/url', 
					    data: {
					        id: this.musicList[i].id
					    },
					    header: {
					        'custom-header': 'playMusicBykeywords'
					    },
					    success: (res) => {
							res.data.data[0].name=this.musicList[i].name;
							this.musicList[i].isAdd = 0;
							this.playList[i]=res.data.data[0];
							this.playUrlList[i]=res.data.data[0].url;
						}
					});
				};
			}
		},
		methods: {
			// 实时获取输入框的值
			onKeyInput: function(event) {
			    this.keyWord = event;
			},
			// 搜索歌曲
			searchMusic() {
				this.listName = '歌曲列表';
				uni.request({
				    url: 'https://autumnfish.cn/search', 
				    data: {
				        keywords: this.keyWord
				    },
				    header: {
				        'custom-header': 'searchMusicBykeywords'
				    },
				    success: (res) => {
						this.musicList = res.data.result.songs;
						for(let i=0;i<this.musicList.length;i++){
							uni.request({
							    url: 'https://autumnfish.cn/song/url', 
							    data: {
							        id: this.musicList[i].id
							    },
							    header: {
							        'custom-header': 'playMusicBykeywords'
							    },
							    success: (res) => {
									res.data.data[0].name=this.musicList[i].name;
									this.musicList[i].isAdd = 0;
									this.playList[i]=res.data.data[0];
									this.playUrlList[i]=res.data.data[0].url;
								}
							});
						};
				    }
				});
				
			},
			// 播放歌曲
			playMusic(id,index) {
				if(this.musicList[index].rtype==0){
					for(let i=0;i<this.musicList.length;i++){
					this.musicList[i].rtype = 0;
					}
					this.musicList[index].rtype = 1;
					this.now=index;
				}else{
					this.musicList[index].rtype=0;
					this.now=-1;
				}
			},
			// 打开弹出层
			addMusic(index){
				this.addSong = this.musicList[index];
				this.addSong.xiabiao = index;
				this.$refs.songsLists.open();
			},
			addMusicToList(index){
				this.songsLists[index].listSongs.push(this.addSong);
				var that = this;
				uni.setStorage({
				    key: 'songsListKey',
				    data: that.songsLists
				});
				this.musicList[that.addSong.xiabiao].isAdd=1;
				this.$refs.songsLists.close();
			},
			// 播放歌单
			playSongsList(index){
				this.listName = this.songsLists[index].listName;
				this.musicList=this.songsLists[index].listSongs;
				for(let i=0;i<this.musicList.length;i++){
					uni.request({
					    url: 'https://autumnfish.cn/song/url', 
					    data: {
					        id: this.musicList[i].id
					    },
					    header: {
					        'custom-header': 'playMusicBykeywords'
					    },
					    success: (res) => {
							res.data.data[0].name=this.musicList[i].name;
							this.musicList[i].isAdd = 2;
							this.playList[i]=res.data.data[0];
							this.playUrlList[i]=res.data.data[0].url;
						}
					});
				};
				this.$refs.playSongsLists.open();
			},
			playMv(index){
				uni.request({
					url:'https://autumnfish.cn/mv/url',
					data:{
						id: this.musicList[index].mvid
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
			}
		},
		watch:{
			now(now){
				for(let i=0;i<this.musicList.length;i++){
				this.musicList[i].rtype = 0;
				}
				this.musicList[now].rtype = 1;
			}
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
	
	.input-box{
		width: 100%;
		height: 60upx;
		font-size: 32upx;
		border: 0;
		border-radius: 60upx;
		-webkit-appearance: none;
		-moz-appearance: none;
		appearance: none;
		padding: 0 3%;
		margin: 0;
		background-color: #ffffff;
	}
	
	.search-box{
		width: 100%;
		background-color: rgb(242, 242, 242);
		padding: 15upx 2.5%;
		display: flex;
		justify-content: space-between;
	}
	
	
	
</style>
