<!-- 滑动切换选项卡+吸顶演示（待完善） -->
<template>
	<view class="content">
		<z-paging ref="paging" @scroll="scroll" :scrollable="scrollable" :hide-empty-view="true"
			:refresher-status.sync="refresherStatus" @query="queryList">
			<!-- 自定义下拉刷新view -->
			<custom-refresher slot="refresher" :status="refresherStatus"></custom-refresher>
			<view class="banner-view" style="height: 250rpx;">
				<view style="font-size: 40rpx;font-weight: 700;">这是一个banner</view>
				<view style="font-size: 24rpx;margin-top: 5rpx;">下方tab滚动时可吸附在顶部</view>
			</view>
			<view class="paging-content" :style="'height:' + pageHeight + 'px'">
				<!-- 小程序中直接修改组件style为position: sticky;无效，需要在组件外层套一层view -->
				<view style="z-index: 100;position: sticky;top :0;">
					<u-tabs-swiper ref="uTabs" :list="list" :current="current" @change="tabsChange" :is-scroll="false"
						swiperWidth="750"></u-tabs-swiper>
				</view>
				<swiper class="swiper" :current="swiperCurrent" @transition="transition"
					@animationfinish="animationfinish">
					<swiper-item class="swiper-item" v-for="(item, index) in list" :key="index">
						<sticky-swiper-item ref="swiperItem" :tabIndex="index" :currentIndex="swiperCurrent"
							@setScrollable="setScrollable">
						</sticky-swiper-item>
					</swiper-item>
				</swiper>
			</view>
		</z-paging>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				refresherStatus: 0,
				scrollTopMap: {},
				// 页面高度
				pageHeight: 0,
				// header高度
				headerHeight: 0,
				scrollable: true,
				list: [{
					name: '测试1'
				}, {
					name: '测试2'
				}, {
					name: '测试3'
				}, {
					name: '测试4'
				}],
				// 因为内部的滑动机制限制，请将tabs组件和swiper组件的current用不同变量赋值
				current: 0, // tabs组件的current值，表示当前活动的tab选项
				swiperCurrent: 0, // swiper组件的current值，表示当前那个swiper-item是活动的
			}
		},
		onLoad() {
			this.pageHeight = uni.getSystemInfoSync().windowHeight;

			this.$nextTick(() => {
				const query = uni.createSelectorQuery();
				query.select('.banner-view').boundingClientRect(data => {
					this.headerHeight = data.height;
				}).exec();
			})
		},
		methods: {
			queryList() {
				//触发了下拉刷新，通过当前tabIndex对应的列表下拉刷新
				this.$refs.swiperItem[this.current].reload(() => {
					this.$refs.paging.complete([]);
				});
			},
			scroll(e) {
				const scrollTop = e.detail.scrollTop;
				//如果当前页面的scroll-view的scrollTop大于等于headerView的高度，则代表吸顶了
				if (scrollTop < this.headerHeight) {
					//还没吸顶
					//禁止子组件的z-paging(scroll-view)滚动，当前页面的z-paging(scroll-view)允许滚动
					this.scrollable = true;
					this.$refs.swiperItem[this.current].setScrollable(false);
				} else {
					//吸顶了
					//允许子组件的z-paging(scroll-view)滚动，当前页面的z-paging(scroll-view)禁止滚动
					this.scrollable = false;
					this.$refs.swiperItem[this.current].setScrollable(true);
				}
			},
			setScrollable(scrollable) {
				this.scrollable = scrollable;
			},
			// tabs通知swiper切换
			tabsChange(index) {
				this.swiperCurrent = index;

			},
			// swiper-item左右移动，通知tabs的滑块跟随移动
			transition(e) {
				let dx = e.detail.dx;
				this.$refs.uTabs.setDx(dx);
			},
			// 由于swiper的内部机制问题，快速切换swiper不会触发dx的连续变化，需要在结束时重置状态
			// swiper滑动结束，分别设置tabs和swiper的状态
			animationfinish(e) {
				let current = e.detail.current;
				this.$refs.uTabs.setFinishCurrent(current);
				this.swiperCurrent = current;
				this.current = current;
				this.$refs.swiperItem[this.current].setScrollable(!this.scrollable);
			}
		}
	}
</script>

<style>
	.banner-view {
		background-color: #007AFF;
		color: white;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.paging-content {
		flex: 1;
		display: flex;
		flex-direction: column;
	}

	.swiper {
		flex: 1;
	}
</style>
