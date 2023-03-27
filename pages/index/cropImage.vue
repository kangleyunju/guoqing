<template>
	<view class="clipperContainer">
		<l-clipper :width="options.width" :scale-ratio="2" :is-lock-width="false" :is-lock-height="false" :height="options.height" :image-url="path" @success="successFn" @cancel="cancel" />
	</view>
</template>
<script>
	export default {
		data() {
			return {
				path: '',
				options: {
					"width": 400,
					"height": 400
				}
			}
		},
		methods: {
			successFn(e) {
				this.getOpenerEventChannel().emit('success', e.url)
				uni.navigateBack()
			},
			cancel() {
				uni.navigateBack()
			}
		},
		onLoad(e) {
			this.path = e.path
			if(e.options){
				this.options = JSON.parse(e.options)
			}
		}
	}
</script>
