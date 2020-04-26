<template>
	<div id="app">
		<Drag class="container" :unit="50" :layout.sync="list">
			<DragItem
				:class="['drag' + item.i]"
				v-for="item in list"
				:key="item.i"
				:layout="item"
			/>
		</Drag>
	</div>
</template>

<script>
import Drag from './components/Drag';
import DragItem from './components/DragItem';
export default {
	name: 'App',
	components: {
		Drag,
		DragItem
	},
	data() {
		return {
			list: [
				{ w: 2, h: 2, i: 1 },
				{ w: 2, h: 2, i: 2 },
				{ w: 2, h: 2, i: 3 },
				{ w: 2, h: 2, i: 4 },
				{ w: 2, h: 2, i: 5 },
				{ w: 2, h: 2, i: 6 }
			]
		};
	}
};
</script>

<style lang="less">
body {
	padding: 0;
	margin: 0;
}
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	background-color: #9e9e9e;
	height: 100vh;
}
</style>
<style lang="less" scoped>
.container {
	width: 500px;
	height: 500px;
	border: 1px solid black;
	position: absolute;
	left: 100px;
	top: 100px;
}
.drag1 {
	background-color: skyblue;
}
.drag2 {
	background-color: blue;
}
.drag3 {
	background-color: palevioletred;
}
.drag4 {
	background-color: palegoldenrod;
}
.drag5 {
	background-color: aqua;
}
.drag6 {
	background-color: violet;
}
</style>
