<template>
	<v-footer dark padless>
		<v-card
			class="grey darken-4 d-flex flex-column flex-grow-1 lighten-1 white--text text-center"
			flat
			tile
		>
			<div class="grey darken-4">
				<v-slider color="green" track-color="grey" always-dirty min="40" max="218"></v-slider>
			</div>

			<div class="pb-5 white--text d-flex flex-row justify-space-between">
				<div>album</div>
				<div class="player__controls">
					<v-icon
						v-for="icon in icons"
						:key="icon.type"
						:class="`player__button--${icon.type}`"
						medium
						:color="style"
						v-bind="icon.size"
					>{{icon.type}}</v-icon>
				</div>
				<div></div>
			</div>
		</v-card>
	</v-footer>
</template>

<script>
import Vue from 'vue';
export default Vue.extend({
	name: 'Player',
	data: () => ({
		style: 'green darken-2',
		icons: [
			{ type: 'replay', size: { medium: 'medium' } },
			{ type: 'skip_previous', size: { large: 'large' } },
			{ type: 'play_circle_filled', size: { 'x-large': 'x-large' } },
			{ type: 'skip_next', size: { large: 'large' } },
			{ type: 'volume_up', size: { medium: 'medium' } },
		],
	}),
});
</script>

<style lang="scss" scoped>
.player__controls {
	width: 50%;

	i {
		cursor: pointer;
	}
}

.player__button--volume_up {
	margin-left: 20px;
}
.player__button--replay {
	margin-right: 20px;
}

div.v-slider__track-container .v-slider--horizontal .v-slider__track-container {
	height: 100% !important;
}
</style>