<template>
	<view class="content">
		<view :style="scrollStyle">
			<movable-area :style="scrollStyle" style="border: 1px solid #F0AD4E;">
				<movable-view
					:x="scrollLeft"
					:y="scrollTop"
					direction="all"
					inertia
					out-of-bounds
					:style="canvasStyle"
					scale
					scale-min="0.5"
					scale-max="2"
					:scale-value="scaleValue"
				>
					<canvas :style="canvasStyle" canvas-id="firstCanvas" id="firstCanvas"></canvas>
					<view
						v-if="pointList[si]"
						class="animation-wrap"
						:style="{
							width: circle_r + 'px',
							height: circle_r + 'px',
							transitionDuration: totalDuration / pointList.length + 's',
							left: pointList[si].xCoordinate - circle_r / 2 + 'px',
							top: pointList[si].yCoordinate - circle_r / 2 + 'px'
						}"
					></view>
				</movable-view>
			</movable-area>
		</view>
	</view>
</template>

<script>
const imgInfo = {
	scaleWidth: 2774 / 1, // 图片缩放初始值
	scaleHeight: 2359 / 1, // 图片缩放初始值
	originWidth: 2774, // 图片原始大小
	originHeight: 2359, // 图片原始大小
	initWidth: 360, // 显示canvas元素的初始值
	initHeight: 260 // 显示canvas元素的初始值
};
/* 动画
	cr:   number  px 多边形半径
	circle_r:   px number 运动圆点半径
	totalList:number 数量 几边形
	totalDuration: number 秒 运动一圈花多长s
	si: number 循环 标记点的数量
	pointList: 获取的坐标点集合
	scrollStyle: 显示canvas元素的样式
	canvasStyle: canvas的样式
	scrollTop: 定位
	scrollLeft: 定位
	ctx: canvas实例
	scaleValue: 缩放值
	*/
export default {
	data() {
		return {
			totalDuration: 4,
			totalList: 2,
			circle_r: 10,
			si: 0,
			pointList: [],
			scrollStyle: {
				width: imgInfo.initWidth + 'px',
				height: imgInfo.initHeight + 'px',
				overflow: 'hidden',
				backgroundColor: '#73b0ff'
			},
			canvasStyle: {
				width: imgInfo.initWidth + 'px',
				height: imgInfo.initWidth + 'px'
			},
			scrollTop: 0,
			scrollLeft: 0,
			bgImgUrl: '/static/logo.png',
			ctx: null,
			scaleValue: 1
		};
	},
	mounted() {
		// #ifdef APP-PLUS
		this.init();
		// #endif
	},
	methods: {
		async getList() {
			this.pointList = [
				{
					xCoordinate: 10,
					yCoordinate: 10
				},
				{
					xCoordinate: 50,
					yCoordinate: 50
				},
				{
					xCoordinate: 150,
					yCoordinate: 50
				},
				{
					xCoordinate: 150,
					yCoordinate: 150
				},
				{
					xCoordinate: 250,
					yCoordinate: 150
				}
			];
		},
		movableScale(e) {
			this.scaleValue = 1;
		},
		async init() {
			try {
				const res = uni.getSystemInfoSync();
				imgInfo.initWidth = res.windowWidth;
				// 获取canvas
				this.ctx = uni.createCanvasContext('firstCanvas');

				this.drawCanvas(imgInfo.initWidth, imgInfo.initHeight);
				this.initTime();
			} catch (e) {
				// error
			}
		},
		initTime() {
			let total = this.pointList.length;
			this.timer = setInterval(res => {
				this.si++;
				if (this.si > total - 1) {
					this.si = total - 1;
					clearInterval(this.timer);
				}
			}, (this.totalDuration * 1000) / total);
		},
		async drawCanvas(width, height) {
			this.ctx.drawImage(this.bgImgUrl, 0, 0, width, height);
			this.ctx.draw();
			// 获取数据
			await this.getList();

			this.ctx.setStrokeStyle('#f56c6c');
			this.ctx.setLineWidth(1);
			
			let arr = [...this.pointList]
			let { xCoordinate, yCoordinate } = arr[0];
			console.log(xCoordinate, yCoordinate)
			this.ctx.moveTo(xCoordinate, yCoordinate);
			// 画线
			let lineArr = arr.slice(1);
			lineArr.forEach(item => {
				this.ctx.lineTo(item.xCoordinate, item.yCoordinate);
			});
			this.ctx.stroke();
			this.ctx.draw(true);
		}
	}
};
</script>

<style lang="scss">
.animation-wrap {
	position: absolute;
	background-color: #ffff00;
	transition-property: left top;
	border-radius: 50%;
	transition-timing-function: linear;
	animation-fill-mode: backwards;
}
.animation-wrap:before {
	content: '';
	display: block;
	width: 14px;
	height: 14px;
	border-radius: 50%;
	opacity: 0.7;
	background-color: #ffff00;
	animation: scaless 1s infinite cubic-bezier(0, 0, 0.49, 1.02);
}
@keyframes scaless {
	0% {
		transform: scale(1);
	}
	50%,
	75% {
		transform: scale(3);
	}
	78%,
	100% {
		opacity: 0;
	}
}
</style>
