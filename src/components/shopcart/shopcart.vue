<template>
	<div>
		<div class="shopcart">
			<div class="content">
				<div class="content-left" @click="toggleList">
					<div class="logo-wrapper">
						<div class="logo" :class="{'highlight':totalCount>0}">
							<i class="icon-shopping_cart"></i>
						</div>
						<div class="num" v-show="totalCount>0">{{totalCount}}</div>
					</div>
					<div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
					<div class="desc">另需配送费￥{{deliveryPrice}}元</div>
				</div>
				<div class="content-right" :class="payClass">
					<div class="pay">
						{{payDesc}}
					</div>
				</div>
			</div>
			<div class="ball-container">
				<div v-for="ball in balls">
					<transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
						<div class="ball" v-show="ball.show">
							<div class="inner inner-hook"></div>
						</div>
					</transition>
				</div>
			</div>
			<transition name="fold">
				<div class="shopcart-list" v-show="listShow">
					<div class="list-header">
						<h1 class="title">购物车</h1>
						<span class="empty" @click="empty">清空</span>
					</div>
					<div class="list-content" ref="cartWrapper">
						<ul>
					  		<li class="food" v-for="food in selectFoods">
								<span class="name">{{food.name}}</span>
								<div class="price">￥{{food.price}}</div>
								<div class="cartcontrol-wrapper">
						  		<cartcontrol :food="food" @add="drop"></cartcontrol>
								</div>
					  		</li>
						</ul>
					</div>
				</div>
			</transition>
		</div>
		<transition name="fade">
			<div class="list-mask" v-show="listShow" @click="toggleList"></div>
		</transition>
	</div>
</template>

<script>
import cartcontrol from '../cartcontrol/cartcontrol.vue';
import BScroll from 'better-scroll';

export default {
	props: {
		selectFoods: {
			type: Array,
			default() {
				return [];
			}
		},
		deliveryPrice: {
			type: Number,
			default: 0
		},
		minPrice: {
			type: Number,
			default: 0
		}
	},
	data() {
		return {
			balls: [
			{
				show: false
			},
			{
				show: false
			},
			{
				show: false
			},
			{
				show: false
			},
			{
				show: false
			}
			],
			dropBalls: [],
			fold: true
		};
	},
	computed: {
		totalPrice() {
			let total = 0;
			this.selectFoods.forEach((food) => {
				total += food.price * food.count;
			});
			return total;
		},
		payDesc() {
			if (this.totalPrice > 0 && this.totalPrice < this.minPrice) {
				let price = this.minPrice - this.totalPrice;
				return `还差￥${price}元起送`;
			} else if (this.totalPrice >= this.minPrice) {
				return '去结算';
			} else {
				return `还差￥${this.minPrice}元起送`;
			}
		},
		totalCount() {
			let count = 0;
			this.selectFoods.forEach((food) => {
				count += food.count;
			});
			return count;
		},
		listShow() {
			if (!this.totalCount) {
				this.fold = true;
				return false;
			}
			let show = !this.fold;
			if (show) {
				this.$nextTick(() => {
					if (!this.scroll) {
						this.scroll = new BScroll(this.$refs.cartWrapper, {
							click: true
						});
					} else {
						this.scroll.refresh();
					}
				});
			}
			return show;
		},
		payClass() {
			if (this.totalPrice < this.minPrice) {
				return 'not';
			} else {
				return 'highlight';
			}
		}
	},
	methods: {
		drop(el) {
			for (let i = 0; i < this.balls.length; i++) {
				let ball = this.balls[i];
				if (!ball.show) {
					ball.show = true;
					ball.el = el;
					this.dropBalls.push(ball);
					return;
				}
			}
		},
		beforeDrop(el) {
			let count = this.balls.length;
			while (count--) {
				let ball = this.balls[count];
				if (ball.show) {
					let rect = ball.el.getBoundingClientRect();
					let x = rect.left - 32;
					let y = -(window.innerHeight - rect.top - 22);
					el.style.display = '';
					el.style.webkitTransform = `translate3d(0,${y}px,0)`;
					el.style.transform = `translate3d(0,${y}px,0)`;
					let inner = el.getElementsByClassName('inner-hook')[0];
					inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
					inner.style.transform = `translate3d(${x}px,0,0)`;
				}
			}
		},
		dropping(el, done) {
			/* eslint-disable no-unused-vars */
			let rf = el.offsetHeight;
			this.$nextTick(() => {
				el.style.webkitTransform = 'translate3d(0,0,0)';
				el.style.transform = 'translate3d(0,0,0)';
				let inner = el.getElementsByClassName('inner-hook')[0];
				inner.style.webkitTransform = 'translate3d(0,0,0)';
				inner.style.transform = 'translate3d(0,0,0)';
				el.addEventListener('transitionend', done);
			});
		},
		afterDrop(el) {
			let ball = this.dropBalls.shift();
			if (ball) {
				ball.show = false;
				el.style.display = 'none';
			}
		},
		empty() {
			this.selectFoods.forEach((food) => {
				food.count = 0;
			});
		},
		toggleList() {
			if (this.totalCount) {
				this.fold = !this.fold;
			}
		}
	},
	components: {
		cartcontrol
	}
};
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin.styl"

		.shopcart
			position:fixed
			left:0
			bottom:0
			z-index:50
			width:100%
			height:48px
			.content
				display:flex
				background:#141d27
				font-size:0
				.content-left
					flex:1
					.logo-wrapper
						z-index:50
						display:inline-block
						position:relative
						top:-10px
						margin:0 12px 0 12px
						padding:6px
						width:56px
						height:56px
						box-sizing:border-box
						vertical-align:top
						border-radius:50%
						background:#141d27
						.logo
							width:100%
							height:100%
							border-radius:50%
							background:#2b343c
							text-align:center
							&.highlight
								background:rgb(0,160,220)
								.icon-shopping_cart
									color:#fff
						.icon-shopping_cart
							color:#80858a
							font-size:24px
							line-height:44px
						.num
							position:absolute
							top:0
							right:0
							width:24px
							height:16px
							line-height:16px
							text-align:center
							border-radius:16px
							font-size:9px
							font-weight:700
							color:#fff
							background:rgb(240,20,20)
							box-shadow:0 4px 8px 0 rgba(0,0,0,0.4)
					.price
						display:inline-block
						vertical-align:top
						line-height:24px
						margin-top:12px
						box-sizing:border-box
						padding-right:12px
						border-right:1px solid rgba(255,255,255,0.1)
						font-size:16px
						font-weight:700
						color:rgba(255,255,255,0.4)
						&.highlight
							color:#fff
					.desc
						display:inline-block
						vertical-align:top
						margin:12px 0 0 12px
						line-height:24px
						font-size:10px
						color:rgba(255,255,255,0.4)
				.content-right
					flex:0 0 105px
					width:105px
					background:#2b343c
					&.highlight
						background:#00b43c
						.pay
							color:#fff
					&.not
						background:#2b343c
					.pay
						height:48px
						line-height:48px
						text-align:center
						font-size:12px
						font-weight:700
						color:rgba(255,255,255,0.4)
		.ball-container
				.ball
					position:fixed
					left:32px
					bottom:22px
					z-index:200
					transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
					.inner
						width:16px
						height:16px
						border-radius:50%
						background:rgb(0,160,220)
						transition:all 0.4s linear					
		.shopcart-list
			position:absolute
			left:0
			top:0
			z-index:-1
			width:100%
			transform: translate3d(0, -100%, 0)
			&.fold-enter-active, &.fold-leave-active
				transition: all 0.4s
			&.fold-enter, &.fold-leave-active
				transform: translate3d(0, 0, 0)
			.list-header
				height:30px
				background:#f3f5f7
				padding:0 18px
				line-height:30px
				border-bottom:1px solid rgba(7,17,27,0.1)
				.title
					float:left
					font-size:14px
					font-weight:200
					color:rbg(7,17,27)
				.empty
					float:right
					font-size:14px
					color:rgb(0,160,220)
			.list-content
				padding:0 18px
				max-height:217px
				overflow:hidden
				background: #fff
				.food
					position:relative
					padding:12px 0
					box-sizing:border-box
					border-1px(rgba(7,17,27,0.1))
					.name
						color:rgb(7,17,27)
						font-size:14px
						line-height:24px
					.price
						position:absolute
						right:100px
						bottom:12px
						line-height:24px
						font-size:14px
						font-weight:700
						color:rgb(240,20,20)
					.cartcontrol-wrapper
						position:absolute
						right:0
						bottom:6px
	.list-mask
		position:fixed
		top:0
		left:0
		width:100%
		height:100%
		z-index:40
		background:rgba(3,3,3,0.4)
		&.fade-enter-active,&.fade-leave-active
			transition:all 0.2s
		&.fade-enter,&.fade-leave-to
			opacity:0 						
</style>
