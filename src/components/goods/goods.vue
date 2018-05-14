<template>
	<div>
		<div class="goods"> 
			<div class="menu-wrapper" ref="menuWrapper">
				<ul>
					<li v-for="(item,index) in goods" class="menu-item" ref="menuList" :class="{'current':currentIndex===index}" @click="selectMenu(index,$event)">
						<span class="text border-1px">
							<span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
						</span>
					</li>
				</ul>
			</div>
			<div class="foods-wrapper" ref="foodsWrapper">
				<ul>
					<li v-for="item in goods" class="food-list" ref="foodList">
						<h1 class="title">{{item.name}}</h1>
						<ul>
							<li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item border-1px">
								<div class="icon">
									<img width="57" height="57" :src="food.icon">
								</div>
								<div class="content">
									<h2 class="name">{{food.name}}</h2>
									<p class="desc">{{food.description}}</p>
									<div class="extra">
										<span class="sell">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
									</div>
									<div class="price">
										<span class="now">￥{{food.price}}</span><span class="past" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
									</div>
									<div class="cartcontrol-wrapper">
										<cartcontrol :food="food" @add="_drop"></cartcontrol>
									</div>
								</div>
							</li>
						</ul>
					</li>
				</ul>
			</div>
			<shopcart ref="shopcart" :selectFoods="selectFoods" :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice"></shopcart>
		</div>
		<food :food="selectedFood" ref="food" @add="_drop"></food>
	</div>
</template>

<script>

import cartcontrol from '../cartcontrol/cartcontrol.vue';
import shopcart from '../shopcart/shopcart.vue';
import BScroll from 'better-scroll';
import food from '../food/food.vue';

const ERR_OK = 0;
const debug = process.env.NODE_ENV !== 'production';

	export default {
		/* eslint-disable */
		props: {
			seller: {
				type: Object
			}
		},
		data() {
			return {
				goods: [],
				listHeight: [],
				scrollY: 0,
				selectedFood: {}
			};
		},
		created() {
			const url = debug ? '/api/goods' : 'http://hanzhibang.cn/sell/api/goods';
			this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
			this.$http.get(url).then((response) => {
				response = response.body;
				if (response.errno === ERR_OK) {
					this.goods = response.data;
					this.$nextTick(() => {
						this._initScroll();
						this._calculateHeight();
					});
				}
			});
		},
		computed: {
			currentIndex() {
				for (let i = 0; i < this.listHeight.length; i++) {
					let listHeight1 = this.listHeight[i];
					let listHeight2 = this.listHeight[i + 1];
					if (!listHeight2 || (this.scrollY >= listHeight1 && this.scrollY < listHeight2)) {
						return i;
					}
				}
				return 0;
			},
			selectFoods() {
				let foods = [];
				this.goods.forEach((good) => {
					good.foods.forEach((food) => {
						if (food.count) {
							foods.push(food);
						}
					});
				});
				return foods;
			}
		},
		methods: {
			_initScroll() {
				this.menuScroll = new BScroll(this.$refs.menuWrapper, {
					click: true
				});
				this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
					click: true,
					probeType: 3
				});
				this.foodsScroll.on('scroll', (pos) => {
					if (pos.y <= 0) {
						this.scrollY = Math.abs(Math.round(pos.y));
					}
				});
			},
			selectMenu(index, event) {
				if (!event._constructed) {
					return;
				}
				let foodList = this.$refs.foodList;
				let el = foodList[index];
				this.foodsScroll.scrollToElement(el, 300);
			},
			selectFood(food, event) {
				if (!event._constructed) {
					return;
				}
				this.selectedFood = food;
				this.$refs.food.show();
			},
			_drop(target) {
			//体验优化，异步执行下落动画
				this.$nextTick(() => {
		  			this.$refs.shopcart.drop(target);
				});
			},
			_calculateHeight() {
				let foodlist = this.$refs.foodList;
				let height = 0;
				this.listHeight.push(height);
				for (let i = 0; i < foodlist.length; i++) {
					let item = foodlist[i];
					height += item.clientHeight;
					this.listHeight.push(height);
				}
			}
		},
		components: {
			shopcart,
			cartcontrol,
			food
		}
	};
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin.styl"


	.goods
		display:flex
		position:absolute
		top:174px
		bottom:46px
		width:100%
		overflow:hidden
		.menu-wrapper
			flex:0 0 80px
			width:80px
			background:#f3f5f7
			.menu-item
				display:table
				height:54px
				width:56px
				line-height:14px
				padding:0 12px 0 12px
				&.current
					position:relative
					z-index:10
					margin-top:-1px
					background:#fff
					font-weight:700
					.text
						border-none()
				.icon
					display:inline-block
					vertical-align:top
					width:12px
					height:12px
					margin-right:2px
					background-size:12px 12px
					background-repeat:no-repeat
					&.decrease
						bg-image('decrease_3')
					&.discount
						bg-image('discount_3')
					&.guarantee
						bg-image('guarantee_3')
					&.invoice
						bg-image('invoice_3')
					&.special
						bg-image('special_3')					
				.text
					display:table-cell
					font-size:12px
					width:56px
					vertical-align:middle
					border-1px (rgba(7,17,27,0.1))
		.foods-wrapper
			flex:1
			.title
				padding-left:14px
				height:26px
				background:#f3f5f7
				border-left:2px solid #d9dde1
				font-sizeL:12px
				color:rgb(147,153,159)
				line-height:26px
			.food-item
				display:flex
				padding:18px 18px 18px 18px
				border-1px (rgba(7,17,27,0.1))
				&:last-child
					border-none()
				.icon
					flex:0 0 57px
					margin-right:10px
				.content
					position:relative
					flex:1
					.name
						line-height:14px
						font-size:14px
						color:rgb(7,17,27)
						margin-top:2px
					.desc,.extra
						margin-top:8px
						color:rgb(147,153,159)
						line-height:10px
						font-size:10px
					.desc
						line-height:12px
					.extra
						:first-child
							margin-right:12px
					.price
						margin-top:8px
						font-weight:700
						line-height:24px
						.now
							color:rgb(240,20,20)
							font-size:14px
							margin-right:8px
						.past
							color:rgb(147,153,159)
							font-size:10px
							text-decoration:line-through
					.cartcontrol-wrapper
						position:absolute
						right:0
						bottom:4px
						height:24px
</style>
