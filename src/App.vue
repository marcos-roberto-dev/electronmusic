<template>
	<v-app>
		<Header />
		<v-content class="black">
			<router-view />
		</v-content>
		<Player />
	</v-app>
</template>

<script lang="ts">
import Vue from 'vue';
import Header from './components/Header.vue';
import Player from './components/Player.vue';

export default Vue.extend({
	name: 'App',
	components: {
		Header,
		Player,
	},
	data: () => ({
		//
	}),
});
</script>
