<template>
	<div
		ref="drag"
		:class="['drag-item', { transition: isTransition }]"
		:style="style"
	>
		<div class="moveblock" @mousedown="handleMouseDown"></div>
	</div>
</template>
<script>
export default {
	name: 'drag-item',

	props: {
		boundary: [Object],
		layout: [Object]
	},
	data() {
		return {
			w: 0,
			h: 0,
			mouse: {
				x0: 0,
				y0: 0
			},
			position: {
				x1: 0,
				y1: 0
			},
			style: {},
			isTransition: false,
			isDrag: false
		};
	},
	mounted() {
		this.$nextTick(() => {
			this.init();
			this.$root.$on('move', (param) => {
				if (!this.isDrag && this.layout.i !== param.i) {
					const thisLeft = this.layout.x;
					const thisRight = this.layout.x + this.layout.w;
					const thisTop = this.layout.y;
					const thisBottom = this.layout.y + this.layout.h;
					const moveLeft = param.x;
					const moveRight = param.x + param.w;
					const moveTop = param.y;
					const moveBottom = param.y + param.h;
					let direction = '';
					let isKnock = false;
					//判断是否在同一列
					if (
						(param.direction === 'up' || param.direction === 'down') &&
						((moveLeft > thisLeft && moveLeft < thisRight) ||
							(moveRight > thisLeft && moveRight < thisRight) ||
							moveLeft === thisLeft ||
							moveRight === thisRight)
					) {
						//判断盒子的相对位置以及是否发生碰撞
						if (moveTop < thisTop) {
							direction = 'down';
							if (moveBottom > thisTop) {
								isKnock = true;
							}
						}
						if (moveBottom > thisBottom) {
							direction = 'up';
							if (moveTop < thisBottom) {
								isKnock = true;
							}
						}
					}
					if (
						(param.direction === 'left' || param.direction === 'right') &&
						((moveTop > thisTop && moveTop < thisBottom) ||
							(moveBottom > thisTop && moveBottom < thisBottom) ||
							moveTop === thisTop ||
							moveBottom === thisBottom)
					) {
						if (moveLeft < thisLeft) {
							direction = 'right';
							if (moveRight > thisLeft) {
								isKnock = true;
							}
						}
						if (moveRight > thisRight) {
							direction = 'left';
							if (moveLeft < thisRight) {
								isKnock = true;
							}
						}
					}
					if (isKnock) {
						let x = this.layout.x;
						let y = this.layout.y;
						if (direction === 'right') {
							x += 1;
						}
						if (direction === 'left') {
							x -= 1;
						}
						if (direction === 'down') {
							y += 1;
						}
						if (direction === 'up') {
							y -= 1;
						}
						this.handleMove({
							x: x,
							y: y,
							w: this.layout.w,
							h: this.layout.h,
							i: this.layout.i,
							direction: direction
						});
					}
				}
			});
		});
	},
	methods: {
		init() {
			this.unit = this.$parent.unit;
			this.$nextTick(() => {
				const x = this.layout.x * this.unit;
				const y = this.layout.y * this.unit;
				this.position.x1 = x;
				this.position.y1 = y;
				this.w = this.layout.w * this.unit;
				this.h = this.layout.h * this.unit;
				this.$set(this.style, 'transform', `translate(${x}px,${y}px)`);
				this.$set(this.style, 'width', `${this.w}px`);
				this.$set(this.style, 'height', `${this.h}px`);
			});
		},
		handleMouseDown(e) {
			this.isDrag = true;
			this.mouse.x0 = e.clientX;
			this.mouse.y0 = e.clientY;
			this.$parent.handleMouseDown(this.layout);
			document.addEventListener('mousemove', this.handleMouseMove);
			document.addEventListener('mouseup', this.handleMouseUp);
		},
		handleMouseMove(e) {
			const x = this.position.x1 + (e.clientX - this.mouse.x0);
			const y = this.position.y1 + (e.clientY - this.mouse.y0);
			this.style['transform'] = `translate(${x}px,${y}px)`;
			const rect = {
				left: x,
				right: x + this.w,
				top: y,
				bottom: y + this.h,
				i: this.layout.i
			};
			this.$parent.handleMove(rect);
		},
		handleMouseUp() {
			this.isDrag = false;
			this.isTransition = true;
			setTimeout(() => {
				this.isTransition = false;
			}, 150);

			this.handleMove({
				x: this.$parent.x / this.unit,
				y: this.$parent.y / this.unit,
				w: this.layout.w,
				h: this.layout.h,
				i: this.layout.i
			});
			setTimeout(() => {
				this.$parent.isShadow = false;
			}, 150);
			document.removeEventListener('mousemove', this.handleMouseMove);
			document.removeEventListener('mouseup', this.handleMouseUp);
		},
		handleMove(param) {
			const x = param.x * this.unit;
			const y = param.y * this.unit;
			if (
				x >= 0 &&
				x + this.w <= this.$parent.W &&
				y >= 0 &&
				y + this.h <= this.$parent.H
			) {
				this.position.x1 = x;
				this.position.y1 = y;
				this.style['transform'] = `translate(${x}px,${y}px)`;
				this.$parent.handlePositionChange({
					x: param.x,
					y: param.y,
					w: param.w,
					h: param.h,
					i: param.i
				});
				this.$root.$emit('move', param);
			}
		},
		handleLockMove() {
			this.lockMove = true;
		}
	}
};
</script>
<style lang="less" scoped>
.drag-item {
	position: absolute;
	z-index: 1;
	&.transition {
		transition: transform 0.15s linear;
	}
	.moveblock {
		position: absolute;
		width: calc(100% - 20px);
		height: 20px;
		left: 0;
		top: 0;
		&:hover {
			cursor: move;
		}
	}
}
</style>
