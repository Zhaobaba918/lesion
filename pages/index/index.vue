<template>
	<view>
		<view class="uni-common-mt">
			<view style="text-align: center; margin-top: 80rpx;">
				<text style="font-size: 60rpx; color: #C8C7CC; ">上传图片进行病变部位检测</text>
			</view>
			<view style="text-align: center; margin-top: 10rpx;">
				<text style="font-size: 30rpx; color: #C8C7CC;">点击图片可以进行预览</text>
			</view>
			<view class="uni-list list-pd">
				<view class="uni-list-cell cell-pd">
					<view class="uni-uploader" style="text-align: center;">
						<image v-if="imageList.length" style="display: inline-block;" :src="imageList[0]" :data-src="imageList[0]" @tap="previewImage"></image>
						<button class="button" @click="chooseImage">添加图片</button>
						
						<view v-if="hasUpLoad" style="text-align: center; margin-top: 30rpx;">
							<view>
								<text style="font-size: 30rpx; color: #C8C7CC;">选择数据集</text>
							</view>
							<uni-data-checkbox style="display: inline-block;" mode="button" v-model="chosenDataSet" :localdata="dataSets"></uni-data-checkbox>
							<!-- <uni-data-checkbox style="display: inline-block;" mode="button" multiple v-model="chosenDataSets" :localdata="dataSets"></uni-data-checkbox> -->
							<view>
								<text style="font-size: 30rpx; color: #C8C7CC;">选择损失函数</text>
							</view>
							<uni-data-checkbox style="display: inline-block;" mode="button" v-model="chosenLoss" :localdata="lossSets"></uni-data-checkbox>
							<!-- <uni-data-checkbox style="display: inline-block;" mode="button" multiple v-model="chosenLoss" :localdata="lossSets"></uni-data-checkbox> -->
							<button class="button" @click="testImage">进行识别</button>
						</view>
					</view>
					<view v-if="hasChecked" style="text-align: center;">
						<view style="text-align: center; margin-top: 60rpx;">
							<text style="font-size: 50rpx; color: #C8C7CC; ">检测结果</text>
						</view>
						<image style="display: inline-block;" :src="resultUrl" :data-src="resultUrl" @tap="previewImage"></image>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>
<script>
	var sourceType = [
		['camera'],
		['album'],
		['camera', 'album']
	]
	var sizeType = [
		['compressed'],
		['original'],
		['compressed', 'original']
	]
	export default {
		data() {
			return {
				title: 'choose/previewImage',
				imageList: [],
				resultUrl:'',
				sourceTypeIndex: 2,
				sourceType: ['拍照', '相册', '拍照或相册'],
				sizeTypeIndex: 2,
				sizeType: ['压缩', '原图', '压缩或原图'],
				hasUpLoad:false,
				hasChecked:false,
				chosenDataSet: 0,
				chosenLoss:0,
				dataSets: [{
									text: 'EX',
									value: 0
								}, {
									text: 'MA',
									value: 1
								}, {
									text: 'MA_PRO',
									value: 2
								}, {
									text: 'SE',
									value: 3
								}, {
									text: 'HE',
									value: 4
								}],
				lossSets:[{
									text: 'BCE',
									value: 0
								}, {
									text: 'Focal',
									value: 1
								}]
				// qrUrl:''
			}
		},
		methods: {
			downLoadImage:function(){
				// 下载检测后的结果
				uni.downloadFile({
				        url: '',
				        success: (res) =>{
				            if (res.statusCode === 200){
				                uni.saveImageToPhotosAlbum({
				                    filePath: res.tempFilePath,
				                    success: function() {
				                        uni.showToast({
				                            title: "保存成功",
				                            icon: "none"
				                        });
				                    },
				                    fail: function() {
				                        uni.showToast({
				                            title: "保存失败",
				                            icon: "none"
				                        });
				                    }
				                });
				            }
				        }
				    })
			},
			getDataSet(index){
				if(index===0){
					return('EX')
				}
				if(index===1){
					return('MA')
				}
				if(index===2){
					return('MA_PRO')
				}
				if(index===3){
					return('SE')
				}
				return('HE')
			},
			getLoss(index){
				if(index===0){
					return('BCE')
				}
				return('Focal')
			},
			testImage:function(){
				//检测图片
				this.upLoadImage();
				this.hasChecked=true;
			},
			upLoadImage:function(){
				//上传图片
				const uploadTask = uni.uploadFile({
							url:`http://100.64.252.67:8001/getAnnotation?dataset=${this.getDataSet(this.chosenDataSet)}&loss=${this.getLoss(this.chosenLoss)}`,
							filePath: this.imageList[0],
							name: 'file',
							formData: {
								'name': 'image'
							},
							success: (res) => {
								console.log(res)
								this.resultUrl = res.data
								this.hasUpLoad = true
								this.hasChecked=true
								uni.showModal({
									title: '',
									content: '检测成功！',
									success: res => {},
									fail: () => {},
									complete: () => {}
								});
							},
						});
				uploadTask.onProgressUpdate((res) => {
							console.log('上传进度' + res.progress);
							console.log('已经上传的数据长度' + res.totalBytesSent);
							console.log('预期需要上传的数据总长度' + res.totalBytesExpectedToSend);
							// 测试条件，取消上传任务。
							if (res.progress > 300) {
								uploadTask.abort();
							}
						});
			},
			chooseImage: async function() {
				//选择图片
				if (this.imageList.length === 1) {
					let isContinue = await this.isFullImg();
					console.log("是否继续?", isContinue);
					if (!isContinue) {
						return;
					}
				}
				uni.chooseImage({
					sourceType: sourceType[this.sourceTypeIndex],
					sizeType: sizeType[this.sizeTypeIndex],
					count: 1,
					success: (res) => {
						this.hasUpLoad=true
						this.imageList = []
						this.imageList = this.imageList.concat(res.tempFilePaths);
						this.hasChecked=false
						uni.showModal({
							title: '',
							content: '添加成功！',
							success: res => {},
							fail: () => {},
							complete: () => {}
						});
					},
					fail: (err) => {
						console.log("err: ",err);
					}
				})
			},
			isFullImg: function() {
				return new Promise((res) => {
					uni.showModal({
						content: "是否清空现有图片？",
						success: (e) => {
							if (e.confirm) {
								this.imageList = [];
								res(true);
							} else {
								res(false)
							}
						},
						fail: () => {
							res(false)
						}
					})
				})
			},
			previewImage: function(e) {
				//预览图片
				console.log(e)
				var current = e.target.dataset.src
				uni.previewImage({
					urls: [current]
				})
			},
		}
	}
</script>

<style>
	.cell-pd {
		padding: 22rpx 30rpx;
	}

	.list-pd {
		margin-top: 30rpx;
	}
	.button{
		width: 80%; border-radius: 60rpx; background-color: #C8C7CC; color: #ffffff; border-color: #007AFF;
		margin-top: 60rpx;
	}
</style>
