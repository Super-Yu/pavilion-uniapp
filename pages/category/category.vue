<template>
	<view class="rf-category">
		<!--顶部搜索导航栏-->
		<rf-search-bar
			@search="toSearch"
			title="扫一扫"
			icon="iconsaomiao"
			:placeholder="hotSearchDefault"
		/>
		<view
			class="category-list"
			@touchmove.stop.prevent="moveHandle"
			v-if="(styleCateType === 'one_two_three_cover' || styleCateType === 'one_two_three_text') && categoryList.length > 0">
			<!-- 左侧分类导航 -->
			<scroll-view scroll-y="true" class="left">
				<view
					class="row"
					v-for="(fItem, fIndex) in categoryList"
					:key="fItem.id"
					:class="[fIndex === showCategoryIndex ? 'on' : '']"
					@tap="showCategory(fIndex)"
				>
					<view class="text">
						<view class="block" :class="'bg-'+themeColor.name"></view>
						{{ fItem.title }}
					</view>
				</view>
			</scroll-view>
			<!--右侧子导航-->
			<scroll-view scroll-y="true" class="right">
				<view
					v-for="(fItem, fIndex) in categoryList"
					:key="fItem.id"
				>
					<view
						v-if="fIndex === showCategoryIndex"
						class="category">
						<view
							v-if="cateTop && cateTop.cover"
							class="banner"
							@tap="indexTopToDetailPage(cateTop)"
						>
							<image :src="cateTop && cateTop.cover" mode="aspectFill" />
						</view>
						<view
							class="box"
							v-for="sItem in fItem.child"
							:key="sItem.id"
						>
							<view class="second-category-text" @tap="navTo(`/pages/product/list?cate_id=${sItem.id}`)">
								{{ sItem.title }}
								<text class="iconfont iconyou"></text>
							</view>
							<view class="list" v-if="sItem.child && sItem.child.length > 0">
								<view
									class="box"
									v-for="tItem in sItem.child"
									:key="tItem.id"
									@tap.stop="navTo(`/pages/product/list?cate_id=${tItem.id}`)"
								>
									<block v-if="styleCateType === 'one_two_three_cover'">
										<image :src="tItem.cover || errorImage"></image>
										<view class="text">{{ tItem.title }}</view>
									</block>
									<text class="category-tag in1line" v-if="styleCateType === 'one_two_three_text'">
										{{ tItem.title }}
									</text>
								</view>
							</view>
							<view v-else class="no-data">
								<i class="iconfont icon404"></i>
								暂无子分类
							</view>
					</view>
						<view class="no-data" v-if="fItem.child.length === 0">
							<i class="iconfont icon404"></i>
							暂无分类
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
		<rf-empty
			:bottom="bottom"
			:info="'暂无产品分类~'"
			v-if="categoryList.length === 0 && !loading"
		></rf-empty>
		<!--加载动画-->
		<rfLoading isFullScreen :active="loading"></rfLoading>
		<!-- 规格-模态层弹窗 -->
		<view
			class="popup spec show"
			v-if="specClass === 'show'"
			@touchmove.stop.prevent="stopPrevent"
			@tap="hideSpec"
		>
			<!-- 遮罩层 -->
			<view class="mask" @tap="hideSpec"></view>
			<view class="layer" @tap.stop="stopPrevent">
				<rf-attr-content
					:product="productDetail"
					@toggle="toggleSpec"
					:showBuyBtn="true"
					></rf-attr-content>
			</view>
		</view>
	</view>
</template>
<script>
/**
 * @des 分类
 *
 * @author stav stavyan@qq.com
 * @date 2020-03-10 18:19
 * @copyright 2019
 */
import { productCate, productList, productDetail, cartItemCount, cartItemCreate } from '@/api/product';
import rfSearchBar from '@/components/rf-search-bar';
import rfAttrContent from '@/components/rf-attr-content';
import { advList } from '@/api/basic';
import { mapMutations } from 'vuex';
export default {
	components: {
		rfSearchBar,
		rfAttrContent
	},
	data() {
		return {
			hotSearchDefault: '请输入关键字', // 搜索默认关键字
			showCategoryIndex: 0, // 一级菜单高亮显示序号
			categoryList: [], // 分类列表
			specClass: 'none', // 规格弹窗
			productDetail: {}, // 商品详情
			search: {},
			cateTop: {}, // 分类通用广告图
			loading: true,
			styleCateType: this.$mSettingConfig.styleCateType,
			errorImage: this.$mAssetsPath.errorImage,
			currentCateId: '',
			categoryProductList: [],
			price: '',
			total_sales: '',
			loadingType: 'more',
			page: 1,
			moneySymbol: this.moneySymbol,
			productLoading: true,
			currentSecondCategoryList: [],
			categoryInvertedIndex: 0
		};
	},
	computed: {
		bottom () {
			let bottom = 0;
			/*  #ifdef H5  */
      bottom = 90;
			/*  #endif */
			return bottom;
		}
	},
	filters: {
		filterTotalSales(val) {
			if (val > 10000) {
				val = `${(val / 10000).toFixed(2)}w`;
			}
			return val;
		}
	},
	// 每次页面显示 执行该方法
	onLoad() {
		this.initData();
	},
	methods: {
    moveHandle() {
    },
    ...mapMutations(['setCartNum']),
		// 监听二级分类变化
		handleCategoryInvertedChange(item, index) {
      this.categoryInvertedIndex = index;
			this.currentCateId = item.id;
			this.price = '';
			this.total_sales = '';
			this.page = 1;
			this.categoryProductList = [];
			this.getProductList();
		},
		stopPrevent() {},
		// 规格弹窗开关
		toggleSpec(row) {
			if (parseInt(row.status, 10) === 0) return;
			if (this.specClass === 'show') {
				this.specClass = 'hide';
				if (row.stock === 0) {
					this.$mHelper.toast('库存不足');
					return;
				}
				if (row.type.toString() === '1') {
					// 加入购物车
					this.handleCartItemCreate(row.skuId, row.cartCount);
				} else if (row.type.toString() === '2') {
					// 立即购买
					const list = {};
					const data = {};
					data.sku_id = row.skuId;
					data.num = row.cartCount;
					if (
						this.productDetail.point_exchange_type.toString() === '2' ||
						this.productDetail.point_exchange_type.toString() === '4' ||
						(this.productDetail.point_exchange_type.toString() === '3' &&
							this.isPointExchange)
					) {
						list.type = 'point_exchange';
					} else {
						list.type = 'buy_now';
					}
					list.data = JSON.stringify(data);
					this.navTo(`/pages/order/create/order?data=${JSON.stringify(list)}`);
				}
				setTimeout(() => {
					this.specClass = 'none';
				}, 250);
			} else if (this.specClass === 'none') {
				this.getProductDetail(row);
			}
		},
		// 添加商品至购物车
		async handleCartItemCreate(skuId, cartCount) {
			await this.$http
				.post(`${cartItemCreate}`, {
					sku_id: skuId,
					num: cartCount
				})
				.then(async () => {
					await this.$http.get(`${cartItemCount}`).then(async r => {
						this.setCartNum(r.data);
					});
					this.$mHelper.toast('添加成功，在购物车等');
				});
		},
		hideSpec() {
			this.specClass = 'hide';
			setTimeout(() => {
				this.specClass = 'none';
			}, 250);
		},
		// 获取产品详情
		async getProductDetail(row) {
			await this.$http
				.get(`${productDetail}`, {
					id: row.id
				})
				.then(async r => {
					this.productDetail = r.data;
					this.specClass = 'show';
				});
		},
		// 跳转至商品列表
		navTo(route) {
			this.$mRouter.push({ route });
		},
		// 数据初始化
		async initData() {
			this.search = uni.getStorageSync('search') || {};
			this.hotSearchDefault = this.search.hot_search_default || '请输入关键字';
			this.categoryList = [];
			this.categoryProductList = [];
			await this.getProductCate();
			await this.initCartItemCount();
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
		// 获取商品分类列表
		async getProductCate(type) {
			await this.$http
				.get(`${productCate}`)
				.then(async r => {
					if (type === 'refresh') {
						uni.stopPullDownRefresh();
					}
					// 获取分类广告图 注：广告图不是一级分类图
					await this.getAdvList();
					// 过滤掉没有二级菜单的一级菜单
					// r.data.forEach(item => {
					// 	if (item.child.length > 0) {
					// 		this.categoryList.push(item);
					// 	}
					// });
					this.categoryList = [
        {
            "id": "955",
            "title": "精选礼物",
            "pid": "0",
            "cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725853_ZuboV95l.png",
            "level": "1",
            "child": [
                {
                    "id": "958",
                    "title": "送给男生礼物",
                    "pid": "955",
                    "cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725853_ZuboV95l.png",
                    "level": "2",
                    "child": [{
						"id": "956",
						"title": "GUCCI手表",
						"pid": "952",
						"cover": "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1600423068398&di=20d9d9244b96b55a365b46ebb9aa9a49&imgtype=0&src=http%3A%2F%2Fci.xiaohongshu.com%2Ffd682f2b-d389-4190-ae0e-4a95fee850b3%40r_640w_640h.jpg",
						"level": "3",
					},{
						"id": "956",
						"title": "黑冰打火机",
						"pid": "952",
						"cover": "https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=3578841656,2869325610&fm=26&gp=0.jpg",
						"level": "3",
					},{
						"id": "956",
						"title": "可乐套装",
						"pid": "952",
						"cover": "https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=109593988,4063484963&fm=26&gp=0.jpg",
						"level": "3",
					},{
						"id": "956",
						"title": "摩托车模型",
						"pid": "952",
						"cover": "https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=1742207833,3827692931&fm=26&gp=0.jpg",
						"level": "3",
					},{
						"id": "956",
						"title": "紫砂内胆保温杯",
						"pid": "952",
						"cover": "https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=1603113833,698758265&fm=26&gp=0.jpg",
						"level": "3",
					},{
						"id": "956",
						"title": "篮球",
						"pid": "952",
						"cover": "https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=1031452982,1709772870&fm=26&gp=0.jpg",
						"level": "3",
					},{
						"id": "956",
						"title": "JavaScript高级程序第四版",
						"pid": "952",
						"cover": "https://mmbiz.qpic.cn/mmbiz_jpg/riafuBXuN2hqREH2nXbxuSCJAKqJ2NWHaWwBfx55blEAAR8ccsZ9Kjqvno5EmXiaF5UGtH0PeGSHRCWQkeWY6HBw/640?wx_fmt=jpeg",
						"level": "3",
					}]
                },
                {
                    "id": "958",
                    "title": "送给女生礼物",
                    "pid": "955",
                    "cover": null,
                    "level": "2",
                    "child": [{
						"id": "956",
						"title": "布偶熊",
						"pid": "952",
						"cover": "https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=2199263900,3868153756&fm=26&gp=0.jpg",
						"level": "3",
					},{
						"id": "956",
						"title": "费列罗巧克力",
						"pid": "952",
						"cover": "https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=1202017806,1511755875&fm=26&gp=0.jpg",
						"level": "3",
					},{
						"id": "956",
						"title": "520鲜花",
						"pid": "952",
						"cover": "https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=3962113670,1136734729&fm=26&gp=0.jpg",
						"level": "3",
					}]
                }
            ]
        },
        {
            "id": "950",
            "title": "告白策划",
            "pid": "0",
            "cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725853_K216a0ZQ.png",
            "level": "1",
            "child": [
                {
                    "id": "959",
                    "title": "生活家用",
                    "pid": "950",
                    "cover": null,
                    "level": "2",
                    "child": []
                },
                {
                    "id": "952",
                    "title": "adfadsf",
                    "pid": "950",
                    "cover": null,
                    "level": "2",
                    "child": [
                        {
                            "id": "956",
                            "title": "测试",
                            "pid": "952",
                            "cover": null,
                            "level": "3",
                            "child": [
                                {
                                    "id": "957",
                                    "title": "测试1",
                                    "pid": "956",
                                    "cover": null,
                                    "level": "4",
                                    "child": []
                                }
                            ]
                        }
                    ]
                }
            ]
        },
        {
            "id": "949",
            "title": "婚礼策划",
            "pid": "0",
            "cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725853_ynHXfOn6.png",
            "level": "1",
            "child": [
                {
                    "id": "931",
                    "title": "美妆护肤",
                    "pid": "949",
                    "cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725853_IM3Js9SS.png",
                    "level": "2",
                    "child": [
                        {
                            "id": "944",
                            "title": "线路",
                            "pid": "931",
                            "cover": null,
                            "level": "3",
                            "child": [
                                {
                                    "id": "945",
                                    "title": "线路",
                                    "pid": "944",
                                    "cover": null,
                                    "level": "4",
                                    "child": []
                                },
                                {
                                    "id": "939",
                                    "title": "海南省",
                                    "pid": "944",
                                    "cover": null,
                                    "level": "4",
                                    "child": [
                                        {
                                            "id": "942",
                                            "title": "12",
                                            "pid": "939",
                                            "cover": null,
                                            "level": "5",
                                            "child": []
                                        },
                                        {
                                            "id": "940",
                                            "title": "三亚市",
                                            "pid": "939",
                                            "cover": null,
                                            "level": "5",
                                            "child": [
                                                {
                                                    "id": "941",
                                                    "title": "天涯海角",
                                                    "pid": "940",
                                                    "cover": null,
                                                    "level": "6",
                                                    "child": []
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "id": "932",
                            "title": "电子商品",
                            "pid": "931",
                            "cover": null,
                            "level": "3",
                            "child": [
                                {
                                    "id": "936",
                                    "title": "cpu",
                                    "pid": "932",
                                    "cover": null,
                                    "level": "4",
                                    "child": [
                                        {
                                            "id": "938",
                                            "title": "迪内",
                                            "pid": "936",
                                            "cover": "http://demo.rageframe.com/attachment/images/2020/08/31/image_1598858602_E2yc5rf3.jpg",
                                            "level": "5",
                                            "child": []
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            ]
        },
        {
            "id": "908",
            "title": "婚宴用品",
            "pid": "0",
            "cover": "http://demo2.rageframe.com/attachment/images/2020/09/10/image_1599725845_TRix3rCW.png",
            "level": "1",
            "child": []
        },
        {
            "id": "961",
            "title": "母婴用品",
            "pid": "0",
            "cover": null,
            "level": "1",
            "child": []
        }
    ];
					let cateId = uni.getStorageSync('indexToCateId');
					if (cateId) {
						this.categoryList.forEach((item, index) => {
							if (cateId === item.id) {
                this.showCategoryIndex = index;
							}
						});
					} else {
						if (this.styleCateType === 'one_two_product' && r.data[0].child.length > 0) {
							this.currentCateId = r.data[0].child[0].id;
						} else {
							this.showCategoryIndex = 0;
							this.currentCateId = this.categoryList && this.categoryList[0].id;
						}
					}
					if (r.data.length > 0) {
						this.currentSecondCategoryList = r.data[0].child;
					}
					this.page = 1;
					this.categoryProductList = [];
					await this.getProductList();
					this.loading = false;
				})
				.catch(() => {
					if (type === 'refresh') {
						uni.stopPullDownRefresh();
					}
					this.loading = false;
				});
		},
		// 获取广告列表
		async getAdvList() {
			await this.$http
				.get(`${advList}`, {
					location: 'cate_top'
				})
				.then(r => {
					this.cateTop = r.data.cate_top[0];
				});
		},
		// 分类切换显示
		showCategory(index) {
			this.showCategoryIndex = index;
		},
		showProduct(item, index) {
			this.categoryInvertedIndex = 0;
			this.currentSecondCategoryList = item.child;
			this.showCategoryIndex = index;
			if (this.styleCateType === 'one_two_product' && item.child.length > 0) {
				this.currentCateId = item.child[0].id;
			} else {
				this.currentCateId = item.id;
			}
			this.price = '';
			this.total_sales = '';
			this.page = 1;
			this.categoryProductList = [];
			this.getProductList();
		},
		handleSalesOrderByChange () {
			if (this.categoryProductList.length === 0) return;
			this.price = '';
      this.total_sales = this.total_sales === 'asc' ? 'desc' : 'asc';
			this.page = 1;
			this.categoryProductList = [];
      this.getProductList();
		},
		handlePriceOrderByChange () {
			if (this.categoryProductList.length === 0) return;
			this.total_sales = '';
      this.price = this.price === 'asc' ? 'desc' : 'asc';
			this.page = 1;
			this.categoryProductList = [];
      this.getProductList();
		},
		// 获取产品列表
		async getProductList() {
			this.productLoading = true;
			const params = {};
			if (this.total_sales) {
				params.total_sales = this.total_sales;
			}
			if (this.price) {
				params.price = this.price;
			}
			params.page = this.page;
			await this.$http
				.get(`${productList}`, {
					cate_id: this.currentCateId,
					...params
				})
				.then(async r => {
					this.productLoading = false;
					this.loadingType = r.data.length === 10 ? 'more' : 'nomore';
					this.categoryProductList = [...this.categoryProductList, ...r.data];
					uni.removeStorageSync('indexToCateId');
				});
		},
		// 跳转至搜索详情页
		toSearch() {
			this.$mRouter.push({
				route: `/pages/index/search/search?data=${JSON.stringify(this.search)}`
			});
		},
		// 跳至广告图指定页面
		indexTopToDetailPage(item) {
			this.$mHelper.handleBannerNavTo(item.jump_type, item.jump_link, item.id);
		},
		getMoreProductList () {
      if (this.loadingType === 'nomore' || this.categoryProductList.length === 0) return;
      this.page++;
      this.getProductList();
		}
	}
};
</script>
<style scoped lang="scss">
page {
	background-color: $color-white;
}
.rf-category {
	/*模块分类*/
	.category-list {
		width: 100%;
		background-color: #fff;
		display: flex;
		.left,
		.right {
			position: absolute;
			top: 100upx;
			/*  #ifdef  APP-PLUS  */
			top: calc(100upx + var(--status-bar-height));
			/*  #endif  */
			bottom: 0upx;
		}

		.left {
			width: 24%;
			left: 0upx;
			background-color: $page-color-base;
			.row {
				width: 100%;
				height: 90upx;
				display: flex;
				align-items: center;

				.text {
					width: 100%;
					position: relative;
					font-size: 28upx;
					display: flex;
					justify-content: center;
					color: #3c3c3c;

					.block {
						position: absolute;
						width: 0upx;
						left: 0;
					}
				}

				&.on {
					height: 100upx;
					background-color: #fff;
					.text {
						font-size: 30upx;
						font-weight: 600;
						color: #2d2d2d;
						.block {
							width: 6upx;
							height: 120%;
							left: 10upx;
						}
					}
				}
			}
		}
		.right {
			width: 76%;
			left: 24%;
			background-color: #fff;
			.second-category {
				z-index: 10;
				white-space: nowrap;
				width: 100%;
				margin: $spacing-sm $spacing-sm $spacing-base;
				.second-category-tag {
					margin-right: $spacing-base;
					padding: 5upx $spacing-lg;
					display: inline-block;
					border-radius: 8upx;
					font-size: $font-sm + 2upx;
				}
				.second-category-normal {
					color: $font-color-dark;
					border: none;
					background-color: $page-color-light;
				}
			}
			.category {
				width: 100%;
				padding: 0 15upx $spacing-base;
				.banner {
					width: 96%;
					margin: 10upx auto 30upx;
					height: 24.262vw;
					border-radius: 10upx;
					overflow: hidden;
					box-shadow: 0upx 5upx 20upx rgba(0, 0, 0, 0.05);
					image {
						width: 100%;
						height: 24.262vw;
					}
				}
				.order-by-wrapper {
					height: 48upx;
					line-height: 48upx;
					display: flex;
					justify-content: space-between;
					margin: $spacing-sm 0;
					font-size: $font-sm + 2upx;
					.order-by {
						display: flex;
						justify-content: center;
						align-items: center;
						.order-by-item {
							display: flex;
							justify-content: center;
							align-items: center;
              font-size: $font-sm + 2upx;
							margin: 0 $spacing-base 0 $spacing-sm;
							color: $font-color-light;
							.iconfont-group {
								margin-top: -2upx;
							}
							.iconfont {
								height: 16upx;
								line-height: 16upx;
								display: block;
								margin-left: 6upx;
								font-weight: 550;
								font-size: $font-sm - 4upx;
							}
						}
					}
				}
				.list {
					margin-top: $spacing-base;
					width: 100%;
					display: flex;
					flex-wrap: wrap;
					.box {
						width: calc(71.44vw / 3);
						margin-bottom: $spacing-lg;
						display: flex;
						justify-content: center;
						align-items: center;
						flex-wrap: wrap;
						image {
							width: 64%;
							height: calc(71.44vw / 3 * 0.64);
						}
						.text {
							margin-top: 8upx;
							width: 100%;
							display: flex;
							justify-content: center;
							font-size: 26upx;
						}
						.category-tag {
							width: 100%;
							display: flex;
							justify-content: center;
							align-items: center;
							padding: $spacing-sm;
							font-size: $font-sm + 2upx;
							margin-right: $spacing-base;
							color: $font-color-dark;
							background-color: $page-color-base;
							border-radius: 6upx;
						}
					}
				}
				.second-category-text {
					display: flex;
					justify-content: space-between;
				}
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
	}
	.one-cover-category {
		margin-top: $spacing-base;
		width: 100%;
		display: flex;
		flex-wrap: wrap;
		.box {
			width: calc(71.44vw / 3);
			margin-bottom: $spacing-lg;
			display: flex;
			justify-content: center;
			align-items: center;
			flex-wrap: wrap;
			image {
				width: 64%;
				height: calc(71.44vw / 3 * 0.64);
			}
			.text {
				margin-top: 8upx;
				width: 100%;
				display: flex;
				justify-content: center;
				font-size: 26upx;
			}
			.category-tag {
				width: 100%;
				display: flex;
				justify-content: center;
				align-items: center;
				padding: $spacing-sm;
				font-size: $font-sm + 2upx;
				margin-right: $spacing-base;
				color: $font-color-dark;
				background-color: $page-color-base;
				border-radius: 6upx;
			}
		}
	}
	.tui-product-container {
		flex: 1;
		margin-right: 10upx;
	}
	.tui-product-container:last-child {
		margin-right: 0;
	}
	.category-product-list {
		margin-bottom: 15upx;
		padding-bottom: 15upx;
		border-bottom: 1upx solid rgba(0, 0, 0, 0.08);
	}
	.category-product-list:last-child {
		border-bottom: none;
		background-color: #007aff;
	}
	.tui-pro-item {
		width: 100%;
		margin-bottom: 20upx;
		background: $color-white;
		box-sizing: border-box;
		border-radius: 12upx;
		overflow: hidden;
		transition: all 0.15s ease-in-out;
	}
	.tui-flex-list {
		display: flex;
		margin-bottom: 1upx !important;
	}
	.tui-pro-img {
		width: 100%;
		display: block;
	}
	.tui-proimg-list {
		width: 160upx;
		height: 160upx !important;
		flex-shrink: 0;
		border-radius: 12upx;
	}
	.tui-pro-content {
		width: 100%;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		box-sizing: border-box;
		padding: 0 20upx;
	}
	.tui-pro-tit {
		color: #2e2e2e;
		font-size: 26upx;
		word-break: break-all;
		overflow: hidden;
		text-overflow: ellipsis;
		display: -webkit-box;
		-webkit-box-orient: vertical;
		-webkit-line-clamp: 2;
	}
	.tui-sale-price {
		font-size: 34upx;
		font-weight: 500;
	}
	.tui-factory-price {
		font-size: 24upx;
		color: #a0a0a0;
		text-decoration: line-through;
		padding-left: 12upx;
	}
	.tui-pro-pay {
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 24upx;
		color: #656565;
		.iconfont {
			font-size: $font-lg + 6upx;
			padding: 0 $spacing-sm;
		}
	}
	.popup .layer {
		// #ifndef H5
		bottom: 0;
		// #endif
	}
}
</style>
