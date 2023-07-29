<template>
	<main class="main">
		<header>
			<div id="advertising">
				
			</div>
		</header>
		<section class="section">
<!-- 			<input class="section-edit" type="text" v-model="shareUrl" placeholder="请输入分享地址" /> -->
						<uni-easyinput class="uni-mt-5 section-edit" suffixIcon="search" v-model="shareUrl" placeholder="请输入分享地址" @iconClick="getInfo"></uni-easyinput>

			<button @click="getInfo()" size="mini" class="download" :loading="loading">解析</button>
			<button @click="getTypes()" size="mini" class="download" :loading="loading">下载</button>
		</section>
		<div>
			<div class="cover">
				<img :src="info.cover" alt="" fit="contain" v-if="info?.cover">
				<img src="../../static/image/SpritecraftOutput.png" v-else>
			</div>
			<div class="desc">
				{{ info?.desc }}
			</div>
		</div>
	</main>
</template>

<script setup>
import { ref, created } from "vue";
import { onLoad, onUnload } from '@dcloudio/uni-app'
import { useStore } from "vuex";

 const store = useStore();
 const advertising = ref()
	onLoad(()=>{
			advertising.value = wx.createInterstitialAd({adUnitId: 'advertising'})
			if(!advertising.value) return
			advertising.value.show()
	})
	onUnload(()=>{
		advertising.value.destory()
	})

	const info = ref()
	// https://v.douyin.com/ix4Vxe9
	const shareUrl = ref('')
	const loading =ref(false)
	const getInfo = () => {
		let url = shareUrl.value.trim();
		if (!url) {
			wx.showToast({
				icon: 'error',
				title: '请输入'
			})
			return
		}
		url = url.replaceAll('#', '')
		url = url.replaceAll('&', '')
		return new Promise((resolve, reject) => {
			loading.value = true
			uni.request({
				url: `https://justinli.love/video/info?url=${url}`,
				method: 'GET',
				success: (res) => {
					console.log('----------res------', res)
					if (!res.statusCode === 200) {
						reject(res.data)
						return
					}
					info.value = res.data
					console.log('info', info)
					resolve(res)
				},
				fail: (err) => {
					console.error('------err------', err)
					reject(err)
				},
				complete: () =>{
					loading.value = false
				}
			})
		})
	}
	
	const options = ref()
	const selectedType = ref()

	const getTypes = () => {
		loading.value = false
		wx.showLoading({
			title: '下载中',
		})
		uni.request({
			url: `https://justinli.love/video/list`,
			method: 'GET',
			success: (res) => {
				if (res.statusCode !== 200) {
					return
				}
				options.value = res.data;
				selectedType.value = res.data[0].value;
				download()
			},
			fail: (err) => {
				loading.value = true
				wx.hideLoading()
				wx.showToast({
					icon: 'error',
					title: '请重试'
				})
				console.error('------err------', err)
			}
		})

	}
	const download = () => {
		let url = shareUrl.value.trim();
		if (!url) {
			wx.showToast({
				icon: 'error',
				title: '请输入'
			})
			return
		}
		url = url.replaceAll('#', '')
		url = url.replaceAll('&', '')
		let filePath = wx.env.USER_DATA_PATH + '/' + new Date().valueOf() + '.mp4'
		wx.downloadFile({
			url: 'https://justinli.love/' + 'video/download?type=' + selectedType.value + '&url=' + url,
			filePath,
			success(res) {
				if (res.statusCode === 200) {
					wx.saveVideoToPhotosAlbum({
						filePath: res.filePath,
						success(res) {
							wx.hideLoading()
							wx.showToast({
								icon: 'success',
								title: '已保存到相册'
							})
						},
						fail(err) {
							wx.hideLoading()
							wx.showToast({
								icon: 'error',
								title: '请重试'
							})
							console.log('err', err)
						},
						complete:()=>{
							loading.value = true
						}
					})
				}
			},
			fail:()=>{
				wx.hideLoading()
				wx.showToast({
					icon: 'error',
					title: '请重试'
				})
				loading.value = true
			}
		})
	}
</script>

<style scoped lang="scss">
	input {
		border: 1px solid #d3d1d8;
	}
	.main{
		padding: 10px 5px;
		.cover {
			width: 300px;
			height: 200px;
			margin: 20px auto 10px;
			image {
				width: 100%;
				height: 100%;
			}
		}
		.desc {
			padding: 10px;
			text-align: center;
		}
		.section{
			display: flex;
			align-items: center;
			flex-direction: column;
		.section-edit {
			width: 80%;
			border: 1px solid #d3d1d8;
			border-radius: 10px;
			padding-left: 10px;
		}
		.download {
			margin-top: 10px;
			border-color: #AEEA10;
			// color: #AEEA10;
			width: 100px
		}
		}
	}
	
</style>