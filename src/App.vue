<template>
	<div id="app">
		<Todo/>
		<Footer/>
	</div>
</template>

<script>
import Todo from "./pages/todolist.vue"
import Footer from "./components/Footer.vue"

export default {
	name: 'App',
	components: {
		Todo,
		Footer
	},
};
</script>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
}
.router-wrapper {
	min-height: 60vh;
}
</style>
