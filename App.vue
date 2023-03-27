<script>
	// #ifndef VUE3
	import Vue from 'vue'
	// #endif
	export default {
		methods:{
			checkIsMobile(){
				setTimeout(()=>{
					let isMobile = navigator.userAgent.match(/(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i)?true:false
					let element=document.querySelector("html")
					if(isMobile){
						element.classList.remove("pc")
					}else{
						element.classList.add("pc")
					}
					Vue.prototype.isMobile = isMobile
				},0)
			}
		},
		onLaunch: function() {
			// #ifdef H5
			if (self == top) {
				this.checkIsMobile()
				window.onresize = () => {
					this.checkIsMobile()
				}
			}
			// #endif
		}
	}
</script>
<style lang="scss">
	@import url("@/static/icon/icon.css");
	html.pc{
		width: 375px;
		background-color: #fff;
		border-radius: 6px;
		margin: 0 auto;
		overflow-y: auto;
		uni-app{
			position: relative;
			height: 100%;
		}
		.uni-page-head{
			top:0;
			left:50%;
			transform: translateX(-50%);
			width: 100%;
		}
		.uni-tabbar-bottom{
			bottom:0;
			left:50%;
			transform: translateX(-50%);
			width: 100%;
		}
		*{
			max-width: 375px;
		}
	}
	.uni-toast{
		padding: 10px 15px;
		.uni-toast__content{
			margin-top: 10px;
		}
	}
	::-webkit-scrollbar{
		display: none;
	}
</style>
