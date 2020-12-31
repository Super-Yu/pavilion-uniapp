<template>
	<view class="cloud">
		<view class="add">
			<image src="https://dss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=2375920824,1229333302&fm=26&gp=0.jpg" mode="widthFix"></image>
		</view>
		<view class="img-content">
			<view class="title">
				云空间数据
			</view>
			<view class="listImg">
				<image class="itemImg" v-for="(item,index) of imgList" :key="index" :src="item" mode="widthFix"></image>
			</view>
			
		</view>
	</view>
</template>

<script>
	export default {
		data(){
			return {
				imgList:[
					'https://dss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=250401372,3249190093&fm=26&gp=0.jpg',
					'https://dss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=250401372,3249190093&fm=26&gp=0.jpg',
					'https://dss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=250401372,3249190093&fm=26&gp=0.jpg',
					'https://dss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=250401372,3249190093&fm=26&gp=0.jpg',
					'https://dss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=250401372,3249190093&fm=26&gp=0.jpg'
				]
			}
		}
	}
</script>

	
<style lang="scss">
.cloud{
	padding: 20upx;
	.add {
		image {
			width: 150upx;
			margin-top:10upx;
			border-radius: 10upx;
		}
		margin-bottom: 40upx;
	}
	
	.img-content{
		background: $color-white;
		border-radius: 10upx;
		padding: 30upx;
		
		.title {
			font-weight: 500;
			font-size: 34upx;
			margin-bottom: 20upx;
			padding-left: 20upx;
			border-left: 6upx #e54d42 solid;
		}
		
		.listImg {
			image {
				width: 49%;
				display: inline-block;
			}
			:nth-child(even) {
				margin-left: 2%;
			}
		}
	}
}
	
</style>
