<template>
  <div class="content-wrap">
    <validation-observer tag="form" v-slot="{ invalid }">
      <form @submit.prevent="handleSubmit(onSubmit)">
        <div class="content" ref="content">
          <div class="blue-top-box">
            <app-step-progress :percent="state.percent" :ing="state.ing" :total="state.total" />
          </div>
          <div class="cont-inner mb-space30">
            <div class="form-box">
              <app-history-taking-item
                v-for="(question, index) in amPmQuestion"
                :key="question.order"
                :question="question"
                :index="(state.ing - 1) * 10 + index + 1"
                v-model="state.amForm[(state.ing - 1) * 10 + index].value"
              />
            </div>
          </div>
        </div>
        <div class="btn-wrap">
          <button type="button" class="btn-line navy" v-show="state.ing !== 1" @click="before">이전</button>
          <button type="button" class="btn-txt navy" v-show="state.ing !== state.total" :disabled="invalid" @click="next">다음</button>
          <button type="button" :disabled="invalid" v-show="state.total === state.ing" class="btn-txt navy" @click="submit">제출</button>
        </div>
      </form>
    </validation-observer>
  </div>
</template>
<route>
{
  "meta" : {
    "title" : "일일 문진"
  }
}
</route>
<script>
import { mapActions } from 'vuex';
import { AppHistoryTakingItem, AppStepProgress } from '@/modules/history/components';
import amList from '@/modules/history/json/amlist.json';
import { initForm, submitForm, TYPE_AM, TYPE_PM, SET_INTERVIEW_LIST } from '@/modules/history';
import { RESPONSE_STATUS } from '@/common/constants';
import _cloneDeep from 'lodash/cloneDeep';
const INIT_STATE = () => ({
  ing: 1,
  total: parseInt(amList.length / 10) + 1,
  percent: 0,
  amForm: initForm(_cloneDeep(amList)),
});

export default {
  data() {
    return {
      state: INIT_STATE(),
    };
  },
  components: { AppHistoryTakingItem, AppStepProgress },
  computed: {
    percent() {
      return parseInt((this.state.ing / this.state.total) * 100);
    },
    amPmQuestion() {
      const start = (this.state.ing - 1) * 10;
      let end = this.state.ing * 10;
      if (end > this.state.amForm.length) {
        end = this.state.amForm.length;
      }
      return this.state.amForm.slice(start, end);
    },
  },
  methods: {
    // 메서드 구현
    ...mapActions({ setInterview: SET_INTERVIEW_LIST }),
    async submit() {
      const formData = submitForm(this.state.amForm);
      const submitData = {
        // interviewType: this.getInterviewType(),
        interviewType: TYPE_AM,
        interviewDate: this.getInterviewDate(),
        answerList: formData,
      };
      const { code, message, data } = await this.setInterview(submitData);
      if (code === RESPONSE_STATUS.SUCCESS) {
        this.$toast('제출되었습니다.');
        this.$router.replace({ name: 'history-taking' });
      }
    },
    // getInterviewType() {
    //   const hour = this.$dayjs().get('h');
    //   return hour < 12 ? TYPE_AM : TYPE_PM;
    // },
    getInterviewDate() {
      const date = this.$dayjs().format('YYYYMMDDhhmm');
      return date;
    },
    changeRequired(nowOrder) {
      const nowQuestion = this.state.amForm.find((element) => {
        return element.order === nowOrder;
      });
      const childQuestion = this.state.amForm.filter((element) => {
        return nowQuestion.child.includes(element.order);
      });
      childQuestion.forEach((child) => {
        if (typeof nowQuestion.value === 'object') {
          nowQuestion.value.forEach((element) => {
            if (child.requiredValues.includes(element + '')) {
              child.answerRequired = 'required';
            } else {
              child.answerRequired = '';
              child.value = '';
            }
          });
        } else {
          if (child.requiredValues.includes(nowQuestion.value)) {
            child.answerRequired = 'required';
          } else {
            child.answerRequired = '';
            child.value = '';
          }
        }
      });
    },
    toTop() {
      this.$refs.content.scrollIntoView();
    },
    before() {
      this.state.ing--;
      this.toTop();
    },
    next() {
      this.state.ing++;
      this.toTop();
    },
  },
  created() {
    this.$eventBus.$on('changeRequired', this.changeRequired);
  },
  beforeDestroy() {
    this.$eventBus.$off('changeRequired');
  },
};
</script>

<style></style>
