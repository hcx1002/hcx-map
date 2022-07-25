<template>
	<div id="app">
		<div style="width: 70%;margin: 20px auto;height: 500px;">
			<hcx-map @addressInfo="addressInfo" @handleSelect="handleSelect"></hcx-map>
		</div>
	</div>
</template>

<script>
	import HcxMap from './components/HcxMap.vue'
	export default {
		name: 'app',
		components: {
			HcxMap
		},
		data() {
			return {

			}
		},
		methods: {
			addressInfo(res) {
				console.log(res);
			},
			handleSelect(res) {
				console.log(res);
			}
		}
	}
</script>

<style>

</style>
