<template>
  <div>
    <div v-if="!orderDone && wizardProgress" v-show="asyncState !== 'pending'">
      <keep-alive>
        <component
          ref="currentStep"
          :is="currentStep"
          @update="processStep"
          :isLastStep="isLastStep"
          :wizardProgress="wizardProgress"
          :wizardData="form"
        ></component>
      </keep-alive>
      <!-- <FormPlanPicker v-if="currentStepNumber === 1" @update="processStep"/> -->
      <!-- <FormUserDetails v-if="currentStepNumber === 2" @update="processStep"/>
    <FormAddress v-if="currentStepNumber === 3" :wizardData="form" @update="processStep"/>
      <FormReviewOrder v-if="currentStepNumber === 4" :wizardData="form" @update="processStep"/>-->
      <div class="progress-bar">
        <div :style="`width: ${progress}%;`"></div>
      </div>

      <!-- Actions -->
      <div class="buttons">
        <button @click="goBack" v-if="currentStepNumber > 1" class="btn-outlined">Back</button>
        <button
          @click="nextButtonAction"
          class="btn"
          :disabled="!moveAllowed"
        >{{isLastStep ? 'Confirm Order' : 'Next'}}</button>
      </div>
      <!-- <pre><code>{{form}}</code></pre> -->
    </div>
    <div v-else>
      <h1 class="title">Thank you !!</h1>
      <h2 class="subtitle">We look forward to shipping your first box!</h2>
      <p class="text-center">
        <a href="https://github.com/instillhubdev" target="_blank" class="btn">Go to InstillHubDev</a>
      </p>
    </div>
    <div class="loading-wrapper" v-if="asyncState === 'pending'">
      <div class="loader">
        <img src="/spinner.svg" alt>
        <p>Please wait we are hitting our servers....</p>
      </div>
    </div>
  </div>
</template>

<script>
import FormPlanPicker from "./FormPlanPicker";
import FormUserDetails from "./FormUserDetails";
import FormAddress from "./FormAddress";
import FormReviewOrder from "./FormReviewOrder";
import { postFormToDB } from "../api";
export default {
  name: "FormWizard",
  components: {
    FormPlanPicker,
    FormUserDetails,
    FormAddress,
    FormReviewOrder
  },
  data() {
    return {
      currentStepNumber: 1,
      moveAllowed: false,
      orderDone: false,
      asyncState : null,
      form: {
        plan: null,
        email: null,
        name: null,
        password: null,
        address: null,
        recipient: null,
        chocolate: false,
        otherTreat: false
      },
      steps: [
        "FormPlanPicker",
        "FormUserDetails",
        "FormAddress",
        "FormReviewOrder"
      ]
    };
  },
  computed: {
    isLastStep() {
      return this.currentStepNumber === this.length;
    },
    wizardProgress() {
      return this.currentStepNumber <= this.length;
    },
    currentStep() {
      return this.steps[this.currentStepNumber - 1];
    },
    length() {
      return this.steps.length;
    },
    progress() {
      return (this.currentStepNumber / this.length) * 100;
    }
  },
  methods: {
    nextButtonAction() {
      if (this.isLastStep) {
        this.submitOrder();
      } else {
        this.goNext();
      }
    },
    async submitOrder() {
      //api call to server
      try {
        this.asyncState = 'pending';
        await postFormToDB(this.form);
        console.log(`form submitted ${this.form}`);
        this.asyncState = 'success';
        this.currentStepNumber++;
        this.orderDone = true;
      } catch (err) {
        console.log(err);
      }
    },
    processStep(step) {
      Object.assign(this.form, step.data);
      this.moveAllowed = step.valid;
    },
    goBack() {
      this.currentStepNumber--;
      this.moveAllowed = true;
    },
    goNext() {
      this.currentStepNumber++;
      this.$nextTick(() => {
        this.moveAllowed = !this.$refs.currentStep.$v.$invalid;
        this.$refs.currentStep.submit();
      });
    }
  }
};
</script>
