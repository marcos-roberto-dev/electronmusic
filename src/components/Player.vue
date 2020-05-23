<template>
	<v-footer dark padless>
		<v-card
			class="grey darken-4 d-flex flex-column flex-grow-1 lighten-1 white--text text-center"
			flat
			tile
		>
			<div class="container-slider grey darken-4 d-flex">
				<span>{{currentTime}}</span>
				<v-slider
					@mousedown="(e) => {handleMouseEnter(e)}"
					@change="handleSlide"
					:value="seek"
					:max="duration"
					color="green"
					track-color="grey"
				></v-slider>
				<span>{{totalTime}}</span>
			</div>

			<div class="pb-5 white--text d-flex flex-row justify-space-between">
				<div>album</div>
				<div class="player__controls">
					<v-icon @click="repeat" class="player__button--replay" medium :color="style">replay</v-icon>
					<v-icon @click="() => skipMusic('previous')" large :color="style">skip_previous</v-icon>
					<v-icon @keypress="playPause" @click="playPause" x-large :color="style">play_circle_filled</v-icon>
					<v-icon @click="() => skipMusic('next')" large :color="style">skip_next</v-icon>
					<v-icon class="player__button--volume_up" medium :color="style">volume_up</v-icon>
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
	loop: boolean;
}

export default Vue.extend({
	name: 'Player',

	data: (): PlayerSound => ({
		style: 'green darken-2',
		change: false,
		loop: false,
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
				this.startTime();
			} else {
				this.clearTime();
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
		initMusic() {
			if (this.change) {
				this.sound = new Howl({ src: this.list[this.index] });
				this.change = false;
			}

			this.sound.play();
			this.sound.on('play', () => {
				this.playing = true;
				this.duration = this.sound.duration();
			});
		},

		resetMusic() {
			this.seek = 0;
			this.timer = '0:00';
			this.playing = false;
		},

		pauseMusic() {
			this.sound.pause();
			this.playing = false;
		},

		stopMusic() {
			this.sound.stop();
			this.playing = false;
		},

		endMusic() {
			this.sound.on('end', () => {
				this.resetMusic();
				if (this.list.length > 1) {
					this.skipMusic('next');
				}

				if (this.loop) {
					this.sound.play();
					this.playing = true;
					this.duration = this.sound.duration();
				}
			});
		},

		skipMusic(change: string) {
			if (change === 'next') this.list.length - 1 > this.index && this.index++;
			if (change === 'previous') this.index >= 1 && this.index--;
			this.resetMusic();
			this.stopMusic();
			this.change = true;
		},

		playPause() {
			if (this.sound.playing()) {
				this.pauseMusic();
			} else {
				this.initMusic();
				this.endMusic();
			}
		},

		startTime() {
			this.intervalWatch = setInterval(() => {
				this.seek++;
				this.timer = this.fancyTimeFormat(this.sound.seek());
			}, 1000);
		},

		clearTime() {
			if (
				typeof this.intervalWatch === 'number' &&
				typeof this.intervalMount === 'number'
			) {
				clearInterval(this.intervalWatch);
				clearInterval(this.intervalMount);
			}
		},

		repeat() {
			this.loop = !this.loop;
		},

		handleMouseEnter() {
			this.playing = false;
		},

		handleMouseLeave() {
			this.playing = true;
		},

		handleSlide(e: any) {
			this.sound.seek(e);
			this.seek = e;
			this.handleMouseLeave();
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

.container-slider {
	padding: 0 20px;
	span {
		margin: 5px 10px 0;
	}
}
</style>