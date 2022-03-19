<template>
  <div class="">
    <div v-if="wizardInProgress" v-show="asyncState !== 'pending'">
      <keep-alive>
        <component
            ref="currentStep"
            :is="currentStep"
            @update="processStep"
            @updateAsyncState="updateAsyncState"
            :wizard-data="form"
        ></component>
      </keep-alive>
      <div class="progress-bar">
        <div :style="`width: ${progress}%;`"></div>
      </div>

      <!-- Actions -->
      <div class="buttons">
        <button
            @click="goBack"
            v-if="currentStepNumber > 1"
            class="btn-outlined"
        >Back
        </button>
        <button
            @click="nextButtonAction"
            :disabled="!canGoNext"
            class="btn"
        >{{ isLastStep ? 'Complete Order' : 'Next'  }}</button>
      </div>

      <pre><code>{{form}}</code></pre>
    </div>
    <div v-else>
      <h1 class="title">THank you!</h1>
      <h2 class="subtitle">
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. A aliquam beatae blanditiis corporis cumque, eius explicabo fugiat illum in nemo, odit quae quas quis sunt tempore temporibus, unde! Commodi, magnam.
      </h2>
    </div>
    <div class="loading-wrapper" v-if="asyncState === 'pending'">
      <div class="loader">
        <img src="/spinner.svg" alt="">
        <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit.</p>
      </div>
    </div>
  </div>

</template>

<script>
import {postFormToDB} from "../api";
import FormPlanPicker from './FormPlanPicker'
import FormUserDetails from './FormUserDetails'
import FormAddress from './FormAddress'
import FormReviewOrder from './FormReviewOrder'
export default {
  name: 'FormWizard',
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder
  },
  data () {
    return {
      canGoNext: false,
      currentStepNumber: 1,
      asyncState: null,
      steps: [
        'FormPlanPicker',
        'FormUserDetails',
        'FormAddress',
        'FormReviewOrder'
      ],
      form: {
        plan: null,
        email: null,
        name: null,
        password: null,
        address: null,
        recipient: null,
        chocolate: false,
        otherTreat: false
      }
    }
  },
  computed: {
    isLastStep() {
      return this.currentStepNumber === this.length
    },
    wizardInProgress() {
      return this.currentStepNumber <= this.length
    },
    currentStep(){
      return this.steps[this.currentStepNumber - 1]
    },
    length(){
      return this.steps.length
    },
    progress () {
      return this.currentStepNumber/this.length * 100
    }
  },
  methods: {
    updateAsyncState(state) {
      this.asyncState = state
    },
    nextButtonAction(){
      if(this.isLastStep) {
        this.submitOrder()
      } else {
        this.goNext()
      }
    },
    submitOrder(){
      this.asyncState = 'pending'
      postFormToDB(this.form)
      .then(() => {
        console.log('form submitted', this.form)
        this.asyncState = 'success'
        this.currentStepNumber++
      })
    },
    processStep(step){
      Object.assign(this.form,step.data)
      this.canGoNext = step.valid
    },
    goBack () {
      this.currentStepNumber--
      this.canGoNext = true
    },
    goNext () {
      this.currentStepNumber++
      // this.canGoNext = false
      this.$nextTick(() => {
        this.canGoNext = !this.$refs.currentStep.$v.$invalid
        // this.$refs.currentStep.submit()
      })

    }
  }
}
</script>
