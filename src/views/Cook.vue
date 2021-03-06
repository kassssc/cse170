<template>
	<div id="cook">
		<div class="is-flex justify-between align-center p-r-sm">
			<div class="is-flex align-center">
				<div class="back-btn" @click="goBackModalActive = true">
					<i class="fas fa-chevron-left"></i>
				</div>
				<h2>Cooking {{ recipe.name }}</h2>
			</div>
			<button class="button green"
							@click="showCommentsTab = !showCommentsTab">
				<i class="fas fa-comment-alt"></i>
			</button>
		</div>

		<div class="columns">
			<div class="column justify-center">
				<div class="steps-nav">
					<button :class="['step-btn', {'current': iscurrStep(0)}]"
									:disabled="timerActive"
									@click="currStep = 0">
						PREP
					</button>
					<button v-for="step in recipe.steps"
									:class="['step-btn', {'current': iscurrStep(step.n)}]"
									:disabled="timerActive"
									@click="onStepClick(step.n)">
						{{ step.n }}
					</button>
				</div>

				<div v-show="iscurrStep(0)" class="content">
					<div class="img-container">
						<div class="img" :style="{'background-image': genURL(recipe.imgURL)}">
						</div>
					</div>
					<div class="ingredient-container">
						<h3>Here's what you need:</h3><br>
						<div v-for="ingredient in recipe.ingredients" class="ingredient-entry">
							<h5 class="m-b-none">{{ ingredient.name }}</h5>
							<div class="ingredient-amount">
								<h5 class="m-b-none m-r-sm">{{ ingredient.quantity }}</h5>
								<h5 class="m-b-none">{{ ingredient.unit }}</h5>
							</div>
						</div>
					</div>

					<button class="start-btn" @click="changeStep(1)">
						START COOKING
					</button>
				</div>

				<div v-for="step in recipe.steps">
					<div v-if="iscurrStep(step.n)" class="content">
						<cooking-step :step="step"
													:lastStep="recipe.steps.length"
													:timerActive.sync="timerActive"
													@next-step="nextStep()"
													@prev-step="prevStep()"
													@timer-done="timerModalActive = true"
													@ignore-timer="ignoreTimerModalActive = true"
													@finish="finishedModalActive = true">
						</cooking-step>
					</div>
				</div>
			</div>

			<div class="comments-tab m-x-md" :class="{'fold': !showCommentsTab}">
				<comments-section :comments="recipe.stepsComments"
													commentBoxPlaceholder="Add a comment...">
				</comments-section>
			</div>
		</div>

		<b-modal :active.sync="skipModalActive"
						 :width="400"
						 :canCancel="['escape', 'outside']">
			<div class="card">
				<div class="card-content">
					<h2 class="color-default m-b-sm text-semibold">Are you sure you want to jump to this step?</h2>
					<div class="is-flex justify-between">
						<button class="button green" @click="skipModalActive = false">
							Stay on Step {{ currStep }}
						</button>
						<button class="button pink" @click="changeStep(stepClicked)">
							Jump to Step {{ stepClicked }}
						</button>
					</div>
				</div>
			</div>
		</b-modal>

		<b-modal :active.sync="finishedModalActive"
						 :width="400"
						 :canCancel="false">
			<div class="card">
				<div class="card-content">
					<h2 class="color-default m-b-sm text-semibold">YAY! Enjoy your dish!</h2>
					<div class="is-flex justify-between">
						<button class="button primary" @click="finishedModalActive = false">
							Back
						</button>
						<router-link to="/home" tag="button" class="button">
							Done Cooking
						</router-link>
					</div>
				</div>
			</div>
		</b-modal>

		<b-modal :active.sync="timerModalActive"
						 :width="350"
						 :canCancel="false">
			<div class="card">
				<div class="card-content">
					<h2 class="color-default m-b-sm text-semibold">Timer is Done!!</h2>
					<div class="is-flex justify-between">
						<button class="button primary" @click="timerModalActive = false">
							Restart Step
						</button>
						<button class="button primary" @click="timerModalNext()">
							Next Step
						</button>
					</div>
				</div>
			</div>
		</b-modal>

		<b-modal :active.sync="ignoreTimerModalActive"
						 :width="350"
						 :canCancel="false">
			<div class="card">
				<div class="card-content">
					<h2 class="color-default m-b-sm text-semibold">You haven't started the timer. Are you sure you want to proceed?</h2>
					<div class="is-flex justify-between">
						<button class="button primary" @click="ignoreTimerModalActive = false">
							Stay in current step
						</button>
						<button class="button primary" @click="timerModalNext()">
							Proceed
						</button>
					</div>
				</div>
			</div>
		</b-modal>

		<b-modal :active.sync="goBackModalActive"
						 :width="450"
						 :canCancel="['escape', 'outside']">
			<div class="card">
				<div class="card-content">
					<h2 class="color-default m-b-sm text-semibold">
						Are you sure you want to stop cooking and exit to the home page?
					</h2>
					<div class="is-flex justify-between">
						<button class="button primary" @click="goBackModalActive = false">
							Continue Cooking
						</button>
						<router-link :to="prevRoute" tag="button" class="button pink">
							Exit to Home Page
						</router-link>
					</div>
				</div>
			</div>
		</b-modal>

	</div>
</template>

<style lang="scss">
</style>

<script>
import _ from 'lodash'
import CommentsSection from '@/components/CommentsSection.vue'
import CookingStep from '@/components/CookingStep.vue'

export default {
	name: 'cook',
	components: {
		CookingStep,
		CommentsSection
	},
	data: function () {
		return {
			currStep: 0,
			finishedModalActive: false,
			skipModalActive: false,
			timerModalActive: false,
			goBackModalActive: false,
			stepClicked: undefined,
			timerActive: false,
			ignoreTimerModalActive: false,
			showCommentsTab: true
		}
	},
	computed: {
		prevRoute: function () {
			let prev = ''
			if (this.$route.params.prev === 'h') {
				prev = 'home'
			} else if (this.$route.params.prev === 's') {
				prev = 'saved'
			} else if (this.$route.params.prev === 'm') {
				prev = 'myrecipes'
			}
			return '/' + prev
		},
		recipe: function() {
			return this.$store.getters.getRecipeById(_.toNumber(this.$route.params.id))
		}
	},
	methods: {
		iscurrStep: function (step) {
			return this.currStep === step
		},
		changeStep: function (n) {
			this.currStep = n
			this.skipModalActive = false;
		},
		prevStep: function () {
			this.currStep -= 1
		},
		nextStep: function () {
			this.currStep += 1
		},
		onStepClick: function (n) {
			this.stepClicked = n
			if (Math.abs(this.currStep - n) > 1) {
				this.skipModalActive = true;
			} else {
				this.changeStep(n)
			}
		},
		timerModalNext: function () {
			this.timerModalActive = false
			this.ignoreTimerModalActive = false
			if (this.currStep < this.recipe.steps.length) {
				this.nextStep()
			} else {
				this.finishedModalActive = true
			}
		}
	},
	created: function () {
	}
}
</script>