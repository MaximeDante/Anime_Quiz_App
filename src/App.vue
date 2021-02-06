<template>
	<main>
		<section class="glass">
			<div v-if="loading" class="spinner-container">
				<div class="spinner">
					<div class="cube1"></div>
					<div class="cube2"></div>
				</div>
			</div>
			<header>
				<div><h1>Anime Quiz</h1></div>
				<div>
					<h1 class="ml3">Ganbarou!</h1>
				</div>
				<div><h1>Ximame</h1></div>
			</header>
			<div class="anime-controls">
				<div class="anime-controls__buttons">
					<button @click.prevent="startQuiz()">START QUIZ</button>
					<button @click="$store.commit('RESETQUIZ')">RESET</button>
				</div>
				<div class="anime-controls__select">
					<label for="difficulty">SELECT DIFFICULTY</label>
					<select v-model="difficulty" class="select">
						<option value="easy">easy</option>
						<option value="medium">medium</option>
						<option value="hard">hard</option>
					</select>
				</div>
			</div>
			<!-- <h1 v-if="loading">Loading...</h1> -->
			<div v-if="Question.length" class="anime-questions">
				<div class="anime-questions__current">
					<h1>
						<span>QUESTION: {{ current + 1 }}</span>
					</h1>
					<p>{{ Question[current].question | replace | apostroph }}</p>
				</div>
				<div class="anime-questions__options">
					<ul v-for="(options, index) in shuffledAnswers" :key="index">
						<li @click="selectedAnswer(index)" :class="checkAnswerClass(index)">
							{{ options | replace | apostroph }}
						</li>
					</ul>
				</div>
				<div class="anime-questions__controls">
					<button @click="submit" :disabled="selectedIndex == null || answered">
						Submit
					</button>
					<button @click="next" :disabled="answered == false">Next</button>
				</div>
				<div>
					<h1>
						Score: <span>{{ correctAnswers }}/{{ Question.length }}</span>
					</h1>
				</div>
			</div>
		</section>
	</main>
</template>

<script>
import { mapState, mapActions } from "vuex";
import anime from 'animejs/lib/anime.es.js';
export default {
	name: "App",
	components: {},
	data() {
		return {
			difficulty: "",
		};
	},
	computed: {
		...mapState([
			"loading",
			"Question",
			"current",
			"selectedIndex",
			"shuffledAnswers",
			"correctIndex",
			"answered",
			"correctAnswers",
		]),
	},
	filters: {
		replace: function(value) {
			return value.replace(/&quot;/g, '"');
		},
		apostroph: function(value) {
			return value.replace(/&#039;/g, "'");
		},
	},
	watch: {
		current: {
			handler() {
				this.$store.commit("RESET");
				this.$store.commit("SHUFFLE_ANSWER");
			},
		},
	},
	mounted() {
		// Wrap every letter in a span
		var textWrapper = document.querySelector(".ml3");
		textWrapper.innerHTML = textWrapper.textContent.replace(
			/\S/g,
			"<span class='letter'>$&</span>"
		);

		anime
			.timeline({ loop: true })
			.add({
				targets: ".ml3 .letter",
				opacity: [0, 1],
				easing: "easeInOutQuad",
				duration: 2250,
				delay: (el, i) => 150 * (i + 1),
			})
			.add({
				targets: ".ml3",
				opacity: 0,
				duration: 1000,
				easing: "easeOutExpo",
				delay: 1000,
			});
	},
	methods: {
		...mapActions(["next", "selectedAnswer", "submit"]),
		startQuiz() {
			this.$store.dispatch("beginQuiz", this.difficulty);
		},
		checkAnswerClass(index) {
			let answerClass = "";
			if (!this.answered && this.selectedIndex === index) {
				answerClass = "selected";
			} else if (this.answered && this.correctIndex === index) {
				answerClass = "correctanswer";
			} else if (
				this.answered &&
				this.selectedIndex === index &&
				this.correctIndex !== index
			) {
				answerClass = "incorrectanswer";
			}
			return answerClass;
		},
	},
};
</script>

<style>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
}
main {
	font-family: "Poppins", sans-serif;
	min-height: 100vh;
	background: linear-gradient(to right top, #65dfc9, #6cdbeb);
	display: flex;
	align-items: center;
	justify-content: center;
}
h1 {
	color: #426696;
	font-weight: 600;
	opacity: 0.8;
}
p {
	color: #658ec6;
	font-weight: 500;
	opacity: 0.8;
}
button {
	background: linear-gradient(to right top, #65dfc9, #6cdbeb);
	border: 2rem #65dfc9;
	color: #426696;
	padding: 0.8rem;
	position: relative;
	cursor: pointer;
	width: 8rem;
	outline: none;
}
button:hover {
	background: linear-gradient(
		to right bottom,
		rgba(255, 255, 255, 0.7),
		rgba(255, 255, 255, 0.3)
	);
}
.glass {
	background: white;
	min-height: 90vh;
	width: 70%;
	background: linear-gradient(
		to right bottom,
		rgba(255, 255, 255, 0.7),
		rgba(255, 255, 255, 0.3)
	);
	border-radius: 2rem;
	z-index: 2;
	backdrop-filter: blur(2rem);
	text-align: center;
}
header {
	display: flex;
	justify-content: space-around;
	padding: 0.5rem 0;
	flex: 33.3%;
	height: 20%;
	background: linear-gradient(
		to right bottom,
		rgba(255, 255, 255, 0.7),
		rgba(255, 255, 255, 0.3)
	);
}
.anime-controls {
	margin-top: 2rem;
}
.anime-controls .anime-controls__buttons {
	display: flex;
	justify-content: center;
}

.anime-controls .anime-controls__buttons button:nth-child(2) {
	margin-left: 1rem;
}
.anime-controls__select {
	margin-top: 1rem;
	display: flex;
	justify-content: center;
}
.anime-controls__select label {
	color: #658ec6;
	font-weight: 500;
	opacity: 0.8;
	margin-right: 0.6rem;
}
.anime-controls__select .select {
	position: relative;
	display: block;
	font-weight: 700;
	color: #426696;
	padding: 0 0.5rem;
	max-width: 100%;
	border: 1px solid #6cdbeb;
	box-shadow: 0 1px 0 1px rgba(0, 0, 0, 0.04);
	border-radius: 0.5em;
	appearance: none;
	outline: none;
	background: linear-gradient(to right top, #65dfc9, #6cdbeb);
}
.anime-questions {
	margin-top: 2rem;
}
.anime-questions__options {
	margin-top: 1rem;
}
ul {
	list-style-type: none;
	padding: 0;
	margin: 0;
	text-align: center;
}
ul li {
	border: 1px solid #ddd;
	margin-top: -1px; /* Prevent double borders */
	background: linear-gradient(
		to right bottom,
		rgba(255, 255, 255, 0.7),
		rgba(255, 255, 255, 0.3)
	);
	padding: 12px;
}
li:hover {
	background: linear-gradient(to right top, #65dfc9, #6cdbeb);
	cursor: pointer;
}
.selected {
	background: skyblue;
}
.correctanswer {
	background: springgreen;
}
.incorrectanswer {
	background: tomato;
	opacity: 1;
}
/* Gamabare Moving letters */
/* Spinner  */
.spinner-container {
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	position: fixed;
	z-index: 100;
}
.spinner {
	/* margin: 100px auto; */
	position: absolute;
	top: 50%;
	left: 50%;
	margin-top: -20px;
	margin-left: -25px;
	width: 50px;
	height: 40px;
	text-align: center;
	font-size: 10px;
}

.cube1,
.cube2 {
	background: linear-gradient(to right top, #65dfc9, #6cdbeb);
	width: 15px;
	height: 15px;
	position: absolute;
	top: 0;
	left: 0;
	-webkit-animation: sk-cubemove 1.8s infinite ease-in-out;
	animation: sk-cubemove 1.8s infinite ease-in-out;
}

.cube2 {
	-webkit-animation-delay: -0.9s;
	animation-delay: -0.9s;
}

@-webkit-keyframes sk-cubemove {
	25% {
		-webkit-transform: translateX(42px) rotate(-90deg) scale(0.5);
	}
	50% {
		-webkit-transform: translateX(42px) translateY(42px) rotate(-180deg);
	}
	75% {
		-webkit-transform: translateX(0px) translateY(42px) rotate(-270deg)
			scale(0.5);
	}
	100% {
		-webkit-transform: rotate(-360deg);
	}
}

@keyframes sk-cubemove {
	25% {
		transform: translateX(42px) rotate(-90deg) scale(0.5);
		-webkit-transform: translateX(42px) rotate(-90deg) scale(0.5);
	}
	50% {
		transform: translateX(42px) translateY(42px) rotate(-179deg);
		-webkit-transform: translateX(42px) translateY(42px) rotate(-179deg);
	}
	50.1% {
		transform: translateX(42px) translateY(42px) rotate(-180deg);
		-webkit-transform: translateX(42px) translateY(42px) rotate(-180deg);
	}
	75% {
		transform: translateX(0px) translateY(42px) rotate(-270deg) scale(0.5);
		-webkit-transform: translateX(0px) translateY(42px) rotate(-270deg)
			scale(0.5);
	}
	100% {
		transform: rotate(-360deg);
		-webkit-transform: rotate(-360deg);
	}
}
</style>
