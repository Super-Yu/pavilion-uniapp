<template>
	<view class="rf-index">
		<!--搜索导航栏-->
		<rf-search-bar
			@search="navToSearch"
			title="扫一扫"
			icon="iconsaomiao"
			:placeholder="hotSearchDefault"
		/>
		<!-- <scroll-view scroll-x class="index-cate" :style="{top: customBar +'px'}" v-if="isOpenIndexCate && categoryList.length > 0">
			<view
				v-for="(item, index) in categoryList"
				:key="index"
				class="index-cate-item"
				:class="tabCurrentIndex === index ? `text-${themeColor.name} active` : ''"
				@tap.stop="tabClick(index, item.id)"
			>
				{{ item.title }}
			</view>
		</scroll-view> -->
		<view :class="isOpenIndexCate && categoryList.length > 0 ? 'main-wrapper' : ''">
			<block v-if="currentCate === 0">
				<!-- 轮播图1 -->
				<view class="swiper">
					<view class="swiper-box">
						<rf-swipe-dot
							:info="carouselList.index_top"
							mode="nav"
							:current="current"
							field="title"
						>
							<swiper @change="handleDotChange" autoplay="true">
								<swiper-item
									v-for="(item, index) in carouselList.index_top"
									@tap="indexTopToDetailPage(item)"
									:key="index"
								>
									<view class="swiper-item">
										<image :src="item.cover" mode="aspectFill" />
									</view>
								</swiper-item>
							</swiper>
						</rf-swipe-dot>
					</view>
				</view>
				<!-- 分类列表 -->
				<!-- <swiper
					:indicator-active-color="themeColor.color"
					indicator-color="#666"
					:indicator-dots="productCateList.length > 10"
					:style="{height: productCateList.length <= 5 ? '200rpx' : '400rpx'}"
					class="category-list-wrapper"
					v-if="productCateList.length > 0">
					<swiper-item
						class="category-list"
						v-for="(fItem, fIndex) in swipeCateList"
						:key="fIndex"
					>
						<view
							class="category"
							v-for="(sItem, sIndex) in fItem" :key="sIndex" @tap.stop="navToCategory(sItem.id)">
							<view class="img">
								<image :src="sItem.cover || errorImage" mode="aspectFill"></image>
							</view>
							<view class="text in1line">{{ sItem.title}}</view>
						</view>
					</swiper-item>
				</swiper> -->
				<!--新闻通知-->
				<!-- <rf-swiper-slide v-if="announceList.length > 0" :list="announceList" class="rf-skeleton" @navTo="navTo('/pages/index/notice/notice')">
					<view slot="header" class="swiper-slide-header">
						<text class="iconfont icongonggao" :class="'text-'+themeColor.name"></text>
					</view>
				</rf-swiper-slide> -->
				<!-- 爆款推荐 -->
				<!-- <view class="hot-recommend">
					<view class="left">
						<image class="hot-recommend-image" @tap="navTo(hotRecommendList[0].url)" :src="hotRecommendList[0].icon"></image>
					</view>
					<view class="right">
						<image class="hot-recommend-image" @tap.stop="navTo(hotRecommendList[1].url)" :src="hotRecommendList[1].icon"></image>
						<image class="hot-recommend-image" @tap.stop="navTo(hotRecommendList[2].url)" :src="hotRecommendList[2].icon"></image>
					</view>
				</view> -->
				<!--新品上市-->
				<!-- <rf-floor-index
					icon="iconxinpin2"
					:list="newProductList"
					@toBanner="indexTopToDetailPage"
					@toList="
					navTo(`/pages/product/list?param=${JSON.stringify({ is_new: 1 })}`)
				"
					:header="{ title: '新品上市', desc: 'New Products Listed' }"
					@detail="navToDetailPage"
					:banner="carouselList.index_new && carouselList.index_new[0]"
				/> -->
				<!--推荐商品-->
				<rf-floor-index
					icon="icontuijian21"
					:list="recommendProductList"
					:header="{ title: '灵魂匹配', desc: '自动为您推荐年龄、兴趣、爱好更加契合的用户' }"
					@toBanner="indexTopToDetailPage"
					@toList="
					navTo(
						`/pages/product/list?param=${JSON.stringify({ is_recommend: 1 })}`
					)
				"
					@detail="navToDetailPage"
					:banner="carouselList.index_recommend && carouselList.index_recommend[0]"
					:bannerShow="false"
				/>
				<!--热门商品-->
				<!-- <rf-floor-index
					icon="iconremen2"
					:list="hotProductList"
					:header="{ title: '热门商品', desc: 'Hot Product' }"
					@toBanner="indexTopToDetailPage"
					@toList="
					navTo(`/pages/product/list?param=${JSON.stringify({ is_hot: 1 })}`)
				"
					@detail="navToDetailPage"
					:banner="carouselList.index_hot && carouselList.index_hot[0]"
				/> -->
				<!--猜您喜欢-->
				<!-- <rf-floor-index
					v-if="guessYouLikeProductList.length > 0"
					icon="iconcainixihuan2"
					:list="guessYouLikeProductList"
					:isLink="false"
					:header="{ title: '猜您喜欢', desc: 'Guess You Like It' }"
					@detail="navToDetailPage"
					:bannerShow="false"
				/> -->
				<!--网站备案号-->
				<!--#ifdef H5-->
				<view class="copyright" v-if="config.web_site_icp">
					{{ config.copyright_desc }}
					<a href="http://www.beian.miit.gov.cn">{{ config.web_site_icp }}</a>
				</view>
				<!-- #endif -->
			</block>
			<view v-else class="index-cate-product-list">
				<rf-product-list :bottom="bottom" :list="categoryProductList"></rf-product-list>
				<rf-load-more
					:status="loadingType"
					v-if="categoryProductList.length > 0"
				></rf-load-more>
				<rf-empty
					:bottom="bottom"
					:info="'暂无该分类产品~'"
					v-if="categoryProductList.length === 0 && !productLoading"
				></rf-empty>
			</view>
		</view>
		<!--页面加载动画-->
		<rfLoading isFullScreen :active="loading"></rfLoading>
		<rf-back-home></rf-back-home>
		<rf-back-top :scrollTop="scrollTop"></rf-back-top>
	</view>
</template>
<script>
/**
 * @des 微商城首页
 *
 * @author stav stavyan@qq.com
 * @date 2020-01-08 14:14
 * @copyright 2019
 */
import {
	indexList,
	productList
} from '@/api/product';
import rfSwipeDot from '@/components/rf-swipe-dot';
import rfFloorIndex from '@/components/rf-floor-index';
import rfSearchBar from '@/components/rf-search-bar';
import rfSwiperSlide from '@/components/rf-swiper-slide';
import rfProductList from '@/components/rf-product-list';
import { mapMutations } from 'vuex';
export default {
	components: {
		rfFloorIndex,
		rfSwipeDot,
		rfProductList,
		rfSearchBar,
		rfSwiperSlide
	},
	data() {
		return {
			current: 0, // 轮播图index
			carouselList: {}, // 广告图
			hotProductList: [], // 热门商品列表
			recommendProductList: [], // 推荐商品列表
			guessYouLikeProductList: [], // 猜您喜欢商品列表
			newProductList: [], // 新品上市商品列表
			productCateList: [], // 商品栏目
			config: {}, // 商户配置
			announceList: [], // 公告列表
			share: {},
			loading: true,
			scrollTop: 0,
			kefuShow: true,
			loadingType: 'more',
			hotSearchDefault: '请输入关键字',
			newsBg: this.$mAssetsPath.newsBg,
			errorImage: this.$mAssetsPath.errorImage,
			appName: this.$mSettingConfig.appName,
			tabCurrentIndex: 0,
			categoryList: [], // 分类列表
			categoryProductList: [], // 分类列表
			page: 1,
			currentCate: 0,
      hotRecommendList: this.$mConstDataConfig.hotRecommendList,
			productLoading: true,
			customBar: this.CustomBar,
			isOpenIndexCate: this.$mSettingConfig.isOpenIndexCate,
			moneySymbol: this.moneySymbol
		};
	},
	onPageScroll(e) {
		this.scrollTop = e.scrollTop;
	},
	onShow() {
		// 初始化购物车数量
		this.setCartNum(uni.getStorageSync('cartNum'));
	},
	onLoad() {
		this.initData();
	},
	computed: {
		// 计算倒计时时间
		second() {
			return function(val) {
				return Math.floor(15 * 60 - (new Date() / 1000 - val));
			};
		},
		bottom () {
			let bottom = 0;
			/*  #ifdef H5  */
      bottom = 90;
			/*  #endif */
			return bottom;
		},
		swipeCateList() {
			const list = this.productCateList;
			let result = [];
			for (let i = 0, len = list.length; i < len; i += 10) {
				result.push(list.slice(i, i + 10));
			}
			return result;
		}
	},
	onShareAppMessage() {
    let shareParams = { title: this.share.share_title || `欢迎来到${this.appName}`, path: '/pages/index/index' };
    return shareParams;
	},
	filters: {
		filterDiscountPrice(val) {
			const price = val.product && (val.product.price * val.discount) / 100;
			switch (val.decimal_reservation_number) {
				case 0:
					return (Math.floor(price * 100) / 100).toFixed(2);
				case 1:
					return (Math.floor(price * 100) / 100).toFixed(0);
				case 2:
					return (Math.floor(price * 100) / 100).toFixed(1);
				default:
					return (Math.floor(price * 100) / 100).toFixed(2);
			}
		},
		filterTotalSales(val) {
			if (val > 10000) {
				val = `${(val / 10000).toFixed(2)}w`;
			}
			return val;
		}
	},
	// 下拉刷新
	onPullDownRefresh() {
		this.getIndexList('refresh');
	},
	// 加载更多
	onReachBottom() {
		if (this.currentCate === 0) return;
		if (this.loadingType === 'nomore') return;
		this.page++;
		this.getProductList(this.currentCate);
	},
	methods: {
		// 顶部tab点击
		tabClick(index, id) {
			this.currentCate = id;
			this.tabCurrentIndex = index;
			if (id === 0) return;
			this.loading = true;
			this.page = 1;
			this.productLoading = true;
			this.categoryProductList = [];
			this.getProductList(id);
		},
		// 获取产品列表
		async getProductList(id) {
			await this.$http
				.get(`${productList}`, {
					cate_id: id,
					page: this.page
				})
				.then(async r => {
					this.loading = false;
					this.productLoading = false;
					this.loadingType = r.data.length === 10 ? 'more' : 'nomore';
					this.categoryProductList = [...this.categoryProductList, ...r.data];
				}).catch(() => {
					this.loading = false;
					this.productLoading = false;
				});
		},
		...mapMutations(['setCartNum']),
		// 监听轮播图切换
		handleDotChange(e) {
			this.current = e.detail.current;
		},
		// 数据初始化
		initData() {
			// 设置购物车数量角标
			this.getIndexList();
			this.initCartItemCount();
		},
		// 设置购物车数量角标
		async initCartItemCount() {
			if (
				this.$mStore.getters.hasLogin &&
				parseInt(uni.getStorageSync('cartNum'), 10) > 0
			) {
				uni.setTabBarBadge({
					index: this.$mConstDataConfig.cartIndex,
					text: uni.getStorageSync('cartNum').toString()
				});
			} else {
				uni.removeStorageSync('cartNum');
				uni.removeTabBarBadge({ index: this.$mConstDataConfig.cartIndex });
			}
		},
		// 通用跳转
		navTo(route) {
			this.$mRouter.push({ route });
		},
		// 跳转至分类模块
		navToCategory(id) {
			if (this.$mSettingConfig.appCateType === '2') {
        uni.setStorageSync('indexToCateId', id);
        this.$mRouter.reLaunch({ route: '/pages/category/category' });
			} else {
        this.navTo(`/pages/product/list?cate_id=${id}`);
			}
		},
		// 通用跳转
		navToSearch() {
			this.$mRouter.push({
				route: `/pages/index/search/search?data=${JSON.stringify(this.search)}`
			});
		},
		// 跳至广告图指定页面
		indexTopToDetailPage(item) {
			this.$mHelper.handleBannerNavTo(item.jump_type, item.jump_link, item.id);
		},
		// 获取主页数据
		async getIndexList(type) {
			await this.$http
				.get(`${indexList}`, {})
				.then(async r => {
					uni.setNavigationBarTitle({ title: this.appName });
					if (type === 'refresh') {
						uni.stopPullDownRefresh();
					}
					// 首页参数赋值
					this.initIndexData({
"search": {
"hot_search_default": "进口小鱼干",
"hot_search_list": [
"狗狼",
"猫薄荷",
"猫抓板",
"小鱼干"
]
},
"adv": {
"index_top": [
{
"id": "41",
"merchant_id": "0",
"title": "最新活动",
"is_title_show": "0",
"cover": "https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=1042855792,2172716422&fm=15&gp=0.jpg",
"location": "index_top",
"silder_text": "",
"start_time": "1595052676",
"end_time": "1601446200",
"jump_link": "",
"jump_type": "product_list_for_cate",
"sort": "0",
"status": "1",
"created_at": "1595052696",
"updated_at": "1599726115",
"view": "904"
}
],
"index_hot": [],
"index_new": [
{
"id": "27",
"merchant_id": "0",
"title": "新品上市",
"is_title_show": "0",
"cover": "http://b2c.rageframe.com/attachment/images/2020/06/30/image_1593522285_wPCKtcCf.png",
"location": "index_new",
"silder_text": "",
"start_time": "1589017848",
"end_time": "1607827800",
"jump_link": "7",
"jump_type": "group_buy_list",
"sort": "5",
"status": "1",
"created_at": "1589017874",
"updated_at": "1599271971",
"view": "1346"
}
],
"index_recommend": []
},
"cate": [
{
"id": "955",
"merchant_id": "0",
"title": "家用电器",
"cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725853_ZuboV95l.png",
"sort": "0",
"level": "2",
"pid": "950",
"tree": "tr_0 tr_950 ",
"index_block_status": "1",
"status": "1",
"created_at": "1599725956",
"updated_at": "1600310608"
},
{
"id": "950",
"merchant_id": "0",
"title": "电脑办公",
"cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725853_K216a0ZQ.png",
"sort": "0",
"level": "1",
"pid": "0",
"tree": "tr_0 ",
"index_block_status": "1",
"status": "1",
"created_at": "1599656805",
"updated_at": "1599725939"
},
{
"id": "949",
"merchant_id": "0",
"title": "手机数码",
"cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725853_ynHXfOn6.png",
"sort": "0",
"level": "1",
"pid": "0",
"tree": "tr_0 ",
"index_block_status": "1",
"status": "1",
"created_at": "1599634238",
"updated_at": "1599725922"
},
{
"id": "908",
"merchant_id": "0",
"title": "第一章",
"cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725845_TRix3rCW.png",
"sort": "0",
"level": "1",
"pid": "0",
"tree": "tr_0 ",
"index_block_status": "1",
"status": "1",
"created_at": "1597800240",
"updated_at": "1600219884"
}
],
"announce": [
{
"id": "1250",
"title": "上线啦",
"cover": "",
"synopsis": "",
"view": "0",
"created_at": "1599119873"
},
{
"id": "883",
"title": "家寻宝贝",
"cover": "http://demo.rageframe.com/attachment/images/2020/08/04/image_1596553513_I6AuD35A.jpg",
"synopsis": "（1）本公告及志愿者提供的寻亲服务均是免费。\r\n（2）如信息需修改、补充等情况，请直接通过本站在线咨询联系客服人员。\r\n（3）本平台不保证登记信息中酬金承诺的有效性，知情人如需要有偿提供线索，请自行与登记人联系确认。 ",
"view": "0",
"created_at": "1596553560"
},
{
"id": "882",
"title": "家寻宝贝",
"cover": "http://demo.rageframe.com/attachment/images/2020/08/04/image_1596553379_r16kbrrn.jpg",
"synopsis": "（1）本公告及志愿者提供的寻亲服务均是免费。\r\n（2）如信息需修改、补充等情况，请直接通过本站在线咨询联系客服人员。\r\n（3）本平台不保证登记信息中酬金承诺的有效性，知情人如需要有偿提供线索，请自行与登记人联系确认。 ",
"view": "0",
"created_at": "1596553422"
},
{
"id": "881",
"title": "宝贝寻亲",
"cover": "http://demo.rageframe.com/attachment/images/2020/08/04/image_1596552707_P33HK2cK.jpg",
"synopsis": "（1）本公告及志愿者提供的寻亲服务均是免费。\r\n（2）如信息需修改、补充等情况，请直接通过本站在线咨询联系客服人员。\r\n（3）本平台不保证登记信息中酬金承诺的有效性，知情人如需要有偿提供线索，请自行与登记人联系确认。 ",
"view": "0",
"created_at": "1596552725"
},
{
"id": "880",
"title": "宝贝寻家",
"cover": "http://demo.rageframe.com/attachment/images/2020/08/04/image_1596552534_qlbclWw2.jpg",
"synopsis": "（1）本公告及志愿者提供的寻亲服务均是免费。\r\n（2）如信息需修改、补充等情况，请直接通过本站在线咨询联系客服人员。\r\n（3）本平台不保证登记信息中酬金承诺的有效性，知情人如需要有偿提供线索，请自行与登记人联系确认。 ",
"view": "0",
"created_at": "1596552550"
},
{
"id": "572",
"title": "守候者",
"cover": "http://demo.rageframe.com/attachment/images/2020/07/28/image_1595932009_JQQOOqAQ.png",
"synopsis": "新功能",
"view": "0",
"created_at": "1593570538"
}
],
"product_hot": [
{
"id": "88",
"name": "PPT代做，不满意不要钱，专业团队服务保证",
"sketch": "",
"keywords": "",
"picture": "https://www.yllook.com/attachment/images/2020/05/18/image_1589789735_UosSpfss.jpg",
"view": "1024",
"match_point": "5.00",
"price": 11,
"market_price": 11,
"cost_price": 11,
"stock": "2",
"total_sales": "57",
"merchant_id": "0",
"shipping_type": "1",
"is_open_presell": "0",
"is_open_commission": "0",
"is_virtual": "1",
"point_exchange_type": "3",
"point_exchange": "200",
"max_use_point": "0",
"integral_give_type": "0",
"give_point": "20",
"unit": "",
"merchant": null,
"commissionRate": 0
},
{
"id": "77",
"name": "华为 HUAWEI 智能体脂秤 体重秤脂肪秤家用健康秤电子秤",
"sketch": "",
"keywords": "",
"picture": "https://www.yllook.com/attachment/images/2020/05/10/image_1589090127_uTioT3ZN.jpg",
"view": "2208",
"match_point": "4.83",
"price": 88,
"market_price": 0,
"cost_price": 50,
"stock": "79991",
"total_sales": "116",
"merchant_id": "0",
"shipping_type": "2",
"is_open_presell": "0",
"is_open_commission": "0",
"is_virtual": "0",
"point_exchange_type": "2",
"point_exchange": "100",
"max_use_point": "0",
"integral_give_type": "1",
"give_point": "5",
"unit": "",
"merchant": null,
"commissionRate": 0
}
],
"product_recommend": [
{
"name": "苏某人",
"picture": "https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=3287653409,3305966191&fm=26&gp=0.jpg",
},{
"name": "郭某人",
"picture": "https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=2841002847,3345892877&fm=26&gp=0.jpg",
},{
"name": "李某人",
"picture": "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1600411724830&di=450637b4553208b83426edf7dea3ea5b&imgtype=0&src=http%3A%2F%2Fimgsrc.baidu.com%2Fforum%2Fw%3D580%2Fsign%3D042d3c9ff2f2b211e42e8546fa816511%2Fe912f995d143ad4bc5061c2c8b025aafa50f06d6.jpg",
},{
"name": "刘某人",
"picture": "https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=2150275148,1763054242&fm=11&gp=0.jpg"
},{
"name": "苏某人",
"picture": "https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=3170716533,472249959&fm=26&gp=0.jpg"
},{
"name": "刘某人",
"picture": "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1600411335814&di=23889e45456c6f17fe0e4305fe6fe0ca&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201810%2F03%2F20181003234106_xldak.jpg" 
}
],
"product_new": [
{
"id": "238",
"name": "【售罄不补】男式无缝休闲运动T恤 夏日清凉运动 吸湿快干透气 - 复制 - 复制",
"sketch": "",
"keywords": "",
"picture": "http://b2c.rageframe.com/attachment/images/2020/06/07/image_1591495392_zTxbOcNE.png",
"view": "7902",
"match_point": "5.00",
"price": 30,
"market_price": 59.9,
"cost_price": 10,
"stock": "996",
"total_sales": "465",
"merchant_id": "0",
"shipping_type": "1",
"is_open_presell": "0",
"is_open_commission": "0",
"is_virtual": "0",
"point_exchange_type": "4",
"point_exchange": "10",
"max_use_point": "0",
"integral_give_type": "1",
"give_point": "50",
"unit": "",
"merchant": null,
"commissionRate": 0
},
{
"id": "90",
"name": "【售罄不补】男式无缝休闲运动T恤 夏日清凉运动 吸湿快干透气",
"sketch": "",
"keywords": "",
"picture": "http://b2c.rageframe.com/attachment/images/2020/06/07/image_1591495392_zTxbOcNE.png",
"view": "7996",
"match_point": "5.00",
"price": 30,
"market_price": 59.9,
"cost_price": 10,
"stock": "996",
"total_sales": "465",
"merchant_id": "0",
"shipping_type": "1",
"is_open_presell": "0",
"is_open_commission": "1",
"is_virtual": "0",
"point_exchange_type": "2",
"point_exchange": "10",
"max_use_point": "0",
"integral_give_type": "1",
"give_point": "50",
"unit": "",
"merchant": null,
"commissionRate": "0.90"
}
],
"product_wholesale": [
{
"id": "40",
"merchant_id": "0",
"merchant_name": "",
"product_id": "60",
"num": "2",
"valid_time": "1",
"type": "1",
"remark": "",
"is_recommend": "1",
"status": "1",
"created_at": "1591149312",
"updated_at": "1591149312",
"merchant": null,
"product": {
"price": 0,
"market_price": 0,
"cost_price": 0,
"wholesale_price": 0
}
}
],
"guess_you_like": [],
"config": {
"web_site_icp": "",
"copyright_companyname": "",
"copyright_url": "",
"copyright_desc": ""
},
"share": {
"share_title": "",
"share_cover": "",
"share_desc": "",
"share_link": ""
}
});
					this.loading = false;
				})
				.catch(() => {
					this.loading = false;
					if (type === 'refresh') {
						uni.stopPullDownRefresh();
					}
				});
		},
		// 首页参数赋值
		initIndexData(data) {
			this.announceList = data.announce || [];
			this.productCateList = data.cate;
			this.categoryList = [{ id: 0, title: '首页' }, ...this.productCateList];
			this.carouselList = data.adv;
			this.search = data.search;
			this.share = data.share;
			uni.setStorageSync('search', this.search);
			this.hotSearchDefault = data.search.hot_search_default || '请输入关键字';
			uni.setStorage({
				key: 'hotSearchDefault',
				data: data.search.hot_search_default
			});
			this.hotProductList = data.product_hot;
			this.recommendProductList = data.product_recommend;
			this.guessYouLikeProductList = data.guess_you_like;
			this.newProductList = data.product_new;
			this.config = data.config;
			this.$mHelper.handleWxH5Share(this.share.share_title || this.appName, this.share.share_desc || `欢迎来到${this.appName}商城`, this.share.share_link || this.$mConfig.hostUrl, this.share.share_cover || this.$mSettingConfig.appLogo);
		},
		// 跳转至商品详情页
		navToDetailPage(data) {
			const { id } = data;
			if (!id) return;
			this.$mRouter.push({ route: `/pages/product/product?id=${id}` });
		},
		// 跳转至分类页
		toCategory() {
			this.$mRouter.switchTab({ route: '/pages/category/category' });
		}
	}
};
</script>
<style lang="scss">
page {
	background-color: $page-color-base;
}
.rf-index {
	background-color: $color-white;
	/*爆款拼团*/
	.wrapper {
		border-radius: 10upx;
		.h-list {
			background-color: $page-color-base;
			white-space: nowrap;
			padding: 0 $spacing-sm;
			.h-item {
				margin: $spacing-sm $spacing-sm $spacing-sm 0;
				display: inline-block;
				background-color: $color-white;
				font-size: $font-sm;
				width: 280upx;
				border-radius: 6upx;
				.h-item-img {
					display: inline-block;
					width: 280upx;
					height: 300upx;
					border-top-left-radius: 12upx;
					border-top-right-radius: 12upx;
					border-bottom: 1upx solid rgba(0, 0, 0, 0.01);
				}
				.title {
					width: 280upx;
					white-space : normal;
					height: 60upx;
					line-height: 30upx;
					font-size: $font-sm;
					padding: 0 $spacing-sm;
					margin: $spacing-sm 0;
				}
				.last-line {
					padding: 0 $spacing-sm $spacing-base;
					margin-bottom: 5upx;
					display: flex;
					justify-content: space-between;
					align-items: center; /* 垂直居中 */
					.red {
						font-size: $font-sm;
						margin-right: 4upx;
					}
				}
				.price {
					font-size: $font-base - 2upx;
					line-height: 1;
					.m-price {
						margin-left: 8upx;
						color: $font-color-light;
						font-size: $font-sm;
						text-decoration: line-through;
					}
				}
			}
		}
	}
	/*轮播图*/
	.swiper {
		width: 100%;
		margin-bottom: 20upx;
		display: flex;
		justify-content: center;
		.swiper-box {
			width: 92%;
			height: 40vw;
			overflow: hidden;
			border-radius: 15upx;
			box-shadow: 0upx 8upx 25upx rgba(0, 0, 0, 0.2);
			//兼容ios，微信小程序
			position: relative;
			z-index: 1;
			swiper {
				width: 100%;
				height: 40vw;
				swiper-item {
					image {
						width: 100%;
						height: 40vw;
					}
				}
			}
		}
	}
  /* 爆款推荐 */
  .hot-recommend {
		background-color: $color-white;
    display: flex;
    padding: $spacing-base $spacing-lg 0;
    .hot-recommend-image {
      width: 100%;
      height: 100%;
    }
    .left {
      flex: 3;
      height: 350upx;
      margin-right: 15upx;
    }
    .right {
      flex: 4;
      .hot-recommend-image {
        height: 170upx;
      }
    }
  }
	/*轮播图2*/
	.swiper-item-text {
		position: absolute;
		bottom: 16upx;
		left: 30upx;
		height: 48upx;
		line-height: 48upx;
		background: rgba(0, 0, 0, 0.2);
		width: 90%;
		color: $color-white;
		border-bottom-left-radius: 12upx;
		padding-left: 20upx;
	}
	/*新闻通知*/
	.swiper-slide-header {
		.iconfont {
			font-size: $font-lg + 8upx;
			font-weight: 600;
		}
	}
	/*分类列表*/
	.category-list-wrapper {
		.category-list {
			width: 100%;
			padding: $spacing-base;
			display: flex;
			justify-content: space-between;
			flex-wrap: wrap;
			.category {
				margin-top: 10upx;
				width: calc(20% - 20upx);
				height: 132upx;
				display: flex;
				text-align: center;
				flex-wrap: wrap;
				.img {
					width: 100%;
					display: flex;
					justify-content: center;
					image {
						width: 12vw;
						height: 12vw;
						border-radius: 50%;
					}
				}
				.text {
					margin-top: 16upx;
					width: 100%;
					display: flex;
					justify-content: center;
					font-size: 24upx;
					color: #3c3c3c;
				}
			}
		}
	}
	/*版权显示*/
	.copyright {
		margin: 10upx 0;
		width: 100%;
		text-align: center;
		color: #666;

		a {
			display: block;
			color: #666;
			text-decoration: none;
		}
	}
	/*限时抢购*/
	.order-item {
		display: flex;
		flex-direction: column;
		background: #fff;
		padding: 0 30upx;
		margin-bottom: 20upx;

		.goods-box-single {
			display: flex;
			padding: 40upx 10upx 10upx;
			height: 330upx;
			border-radius: 12upx;
			margin-top: 20upx;
			box-shadow: 2px 2px 10px rgba(66, 135, 193, 0.2); // 阴影
			border-bottom: 1px solid rgba(0, 0, 0, 0.05); // 边框
			position: relative;

			.goods-img {
				display: block;
				border-radius: 12upx;
				width: 190upx;
				height: 200upx;
			}

			.right {
				flex: 1;
				display: flex;
				flex-direction: column;
				padding: 0 30upx 0 24upx;
				overflow: hidden;

				.title {
					font-weight: bold;
					line-height: 1.2;
					margin: 10upx 0;
				}

				.attr-box {
					line-height: 1.2;
					margin-bottom: 8upx;
					margin-left: 10upx;
				}

				.last-line {
					margin-top: 3upx;
					display: flex;
					justify-content: space-between;
					align-items: center; /* 垂直居中 */
					.red {
						margin-right: 4upx;
					}
				}

				.price {
					font-size: $font-lg;
					line-height: 1;
					.m-price {
						margin-left: 8upx;
						color: $font-color-light;
						font-size: $font-sm;
						text-decoration: line-through;
					}
				}

				.triangle-wrapper {
					position: absolute;
					overflow: hidden;
					top: 0;
					right: 0;
					border-radius: 12upx;

					.triangle {
						color: #5eba8f;
						width: 0;
						height: 0;
						border-top: 120upx solid;
						opacity: 0.8;
						border-left: 120upx solid transparent;
					}

					.triangle-content {
						position: absolute;
						top: 28upx;
						right: 0;
						transform: rotate(45deg);
						font-size: $font-sm - 2upx;
						color: #fff;
					}
				}
			}
		}

		.action-box {
			display: flex;
			justify-content: flex-end;
			align-items: center;
			height: 90upx;
			position: relative;

			.discount-time {
				font-size: $font-sm;
				color: $uni-color-success;
				margin-right: 20upx;
			}
		}
	}
	.index-cate {
		white-space: nowrap;
		z-index: 10;
		background-color: $color-white;
		/*#ifndef MP*/
		position: fixed;
		/*#endif*/
		margin-bottom: $spacing-sm;
		.index-cate-item {
			display: inline-block;
			height: 75upx;
			line-height: 75upx;
			margin: 0 15upx;
			text-align: center;
			width: 120upx;
			font-size: $font-base;
		}
		.active {
			font-weight: 500;
			border-bottom: 2px solid;
		}
	}
	.main-wrapper {
		margin-top: 85upx;
		/*#ifdef MP*/
		margin-top: 0;
		/*#endif*/
	}
	.index-cate-product-list {
		padding-top: $spacing-sm;
		background-color: $page-color-base;
		.no-data {
			margin: 48upx 0;
			color: $font-color-light;
			display: flex;
			justify-content: center;
			align-items: center;
			.iconfont {
				margin-right: 20upx;
			font-size: $font-lg + 16upx;
			}
		}
	}
}
</style>
