<template>
	<v-footer dark padless>
		<v-card
			class="grey darken-4 d-flex flex-column flex-grow-1 lighten-1 white--text text-center"
			flat
			tile
		>
			<div class="grey darken-4">
				<v-slider
					@mousedown="(e) => {mouseEnter(e)}"
					@change="handleSlide"
					:value="seek"
					:max="duration"
					color="green"
					track-color="grey"
				></v-slider>
			</div>

			<div class="pb-5 white--text d-flex flex-row justify-space-between">
				<div>album</div>
				<div class="player__controls">
					<span>{{currentTime}}</span>
					<v-icon class="player__button--replay" medium :color="style">replay</v-icon>
					<v-icon @click="() => skipMusic('previous')" large :color="style">skip_previous</v-icon>
					<v-icon @keypress="playPause" @click="playPause" x-large :color="style">play_circle_filled</v-icon>
					<v-icon @click="() => skipMusic('next')" large :color="style">skip_next</v-icon>
					<v-icon class="player__button--volume_up" medium :color="style">volume_up</v-icon>
					<span>{{totalTime}} {{playing}}</span>
				</div>
				<div></div>
			</div>
		</v-card>
	</v-footer>
</template>

<script lang="ts">
import Vue from 'vue';
import { Howl, Howler } from 'howler';

interface PlayerSound {
	style: string;
	sound: Howl;
	playing: boolean;
	timer: string;
	duration: number;
	seek: number;
	intervalWatch: number | NodeJS.Timeout;
	intervalMount: number | NodeJS.Timeout;
	list: any[];
	index: number;
	change: boolean;
}

export default Vue.extend({
	name: 'Player',

	data: (): PlayerSound => ({
		style: 'green darken-2',
		change: false,
		index: 0,
		list: [
			require('../assets/musics/sound.mp3'),
			require('../assets/musics/sound2.mp3'),
		],
		sound: new Howl({
			src: [require('../assets/musics/sound.mp3')],
		}),
		playing: false,
		timer: '0:00',
		duration: 0,
		seek: 0,
		intervalWatch: 0,
		intervalMount: 0,
	}),
	watch: {
		index() {
			this.playPause();
		},
		playing(isPlay) {
			if (isPlay) {
				this.intervalWatch = setInterval(() => {
					this.seek++;
					this.timer = this.fancyTimeFormat(this.sound.seek());
				}, 1000);
			} else {
				if (
					typeof this.intervalWatch === 'number' &&
					typeof this.intervalMount === 'number'
				) {
					clearInterval(this.intervalWatch);
					clearInterval(this.intervalMount);
				}
			}
		},
	},

	computed: {
		totalTime(): string {
			return this.fancyTimeFormat(this.sound.duration());
		},

		currentTime(): string {
			return this.timer;
		},
	},

	methods: {
		skipMusic(change: string) {
			if (change === 'next') this.list.length - 1 > this.index && this.index++;
			if (change === 'previous') this.index >= 1 && this.index--;
			this.seek = 0;
			this.timer = '0:00';
			this.playing = false;
			this.sound.pause();
			this.change = true;
		},
		mouseEnter(event: any) {
			this.playing = false;
		},

		mouseLeave(event: any) {
			this.playing = true;
		},

		handleSlide(e: any) {
			this.sound.seek(e);
			this.seek = e;
			this.mouseLeave(e);
		},

		playPause() {
			if (this.sound.playing()) {
				this.sound.pause();
				this.playing = false;
			} else {
				if (this.change) {
					this.sound = new Howl({ src: this.list[this.index] });
					this.change = false;
				}
				this.sound.play();
				this.sound.on('play', () => {
					this.playing = true;
					this.duration = this.sound.duration();
				});
				this.sound.on('end', () => {
					this.seek = 0;
					this.timer = '0:00';
					this.playing = false;
				});
			}
		},

		fancyTimeFormat(time: number | Howl) {
			let ret = '';
			if (typeof time === 'number') {
				const hrs = ~~(time / 3600);
				const mins = ~~((time % 3600) / 60);
				const secs = ~~time % 60;

				if (hrs > 0) {
					ret += '' + hrs + ':' + (mins < 10 ? '0' : '');
				}

				ret += '' + mins + ':' + (secs < 10 ? '0' : '');
				ret += '' + secs;

				return ret;
			}
			return ret;
		},
	},
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