<template>
	<div class="drag-container">
		<slot></slot>
		<div
			ref="shadow"
			v-show="isShadow"
			class="shadow"
			:style="{
				transform: `translate(${x}px,${y}px)`,
				height: `${h}px`,
				width: `${w}px`
			}"
		></div>
	</div>
</template>
<script>
export default {
	name: 'drag',
	props: {
		boundary: [Object],
		layout: [Array],
		unit: {
			default: 10
		}
	},
	data() {
		return {
			isShadow: false,
			i: 0,
			W: 0,
			H: 0,
			x: 0,
			y: 0,
			x1: 0,
			y1: 0,
			h: 0,
			w: 0,
			center: {
				x: 0,
				y: 0
			}
		};
	},
	created() {},
	mounted() {
		this.$nextTick(() => {
			this.init();
		});
	},
	methods: {
		init() {
			const containerHeight = this.$el.clientHeight;
			const containerWidth = this.$el.clientWidth;
			if (containerHeight * containerWidth) {
				this.W = containerWidth;
				this.H = containerHeight;
			} else if (this.boundary) {
				this.W = this.boundary.w;
				this.H = this.boundary.h;
			}
			const layout = [...this.thisLayout];
			layout.forEach((item, index) => {
				if (index === 0) {
					item.x = 0;
					item.y = 0;
				} else {
					let y = layout[index - 1].y + layout[index - 1].h;
					let x = 0;
					if ((y + item.h) * this.unit > this.H) {
						y = 0;
						x = layout[index - 1].x + layout[index - 1].w;
					} else {
						x = layout[index - 1].x;
					}
					item.x = x;
					item.y = y;
				}
			});
		},
		handleMouseDown(param) {
			this.x = param.x * this.unit;
			this.y = param.y * this.unit;
			this.h = param.h * this.unit;
			this.w = param.w * this.unit;
			this.i = param.i;
			this.x1 = this.x;
			this.y1 = this.y;
			this.$nextTick(() => {
				this.isShadow = true;
				this.getCenter();
			});
		},
		handleMove(param) {
			const centerX = (param.right - param.left) / 2 + param.left;
			const centerY = (param.bottom - param.top) / 2 + param.top;
			const x = centerX - this.center.x;
			const y = centerY - this.center.y;
			const up = this.y - this.unit;
			const down = this.y + this.unit + this.h;
			const left = this.x - this.unit;
			const right = this.x + this.unit + this.w;
			if (Math.abs(x) > this.unit / 2) {
				if (x < 0 && left >= 0) {
					this.move('left', param.i);
				} else if (x > 0 && right <= this.W) {
					this.move('right', param.i);
				}
			}
			if (Math.abs(y) > this.unit / 2) {
				if (y < 0 && up >= 0) {
					this.move('up', param.i);
				} else if (y > 0 && down <= this.H) {
					this.move('down', param.i);
				}
			}
		},
		move(direction, i) {
			if (direction === 'up') {
				this.y -= this.unit;
			} else if (direction === 'down') {
				this.y += this.unit;
			} else if (direction === 'left') {
				this.x -= this.unit;
			} else if (direction === 'right') {
				this.x += this.unit;
			}
			this.$nextTick(() => {
				this.$root.$emit('move', {
					x: this.x / this.unit,
					y: this.y / this.unit,
					w: this.w / this.unit,
					h: this.h / this.unit,
					i: i,
					direction: direction
				});
			});
			this.getCenter();
		},
		handlePositionChange(val) {
			setTimeout(() => {
				const layout = [];
				this.thisLayout.forEach((el) => {
					let item = el;
					if (item.i === val.i) {
						item = val;
					}
					layout.push(item);
				});
				this.thisLayout = layout;
			}, 1);
		},
		getCenter() {
			this.$nextTick(() => {
				const rect = {
					left: this.x,
					right: this.x + this.w,
					top: this.y,
					bottom: this.y + this.h
				};
				this.center.x = (rect.right - rect.left) / 2 + rect.left;
				this.center.y = (rect.bottom - rect.top) / 2 + rect.top;
			});
		}
	},
	computed: {
		thisLayout: {
			get() {
				return this.layout;
			},
			set(val) {
				this.$emit('update:layout', val);
			}
		}
	}
};
</script>
<style lang="less" scoped>
.drag-container {
	.shadow {
		position: absolute;
		background-color: pink;
		opacity: 0.4;
		z-index: 0;
		transition: all linear 0.1s;
	}
}
</style>
