<template>
  <div class="tab-box">
    <ul class="tab" :class="scrollClass">
      <li v-for="(tab, index) in tabs" :key="index" :class="{ selected: currentTab === index }">
        <button type="button" @click="onClick(index)">{{ tab }}</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: {
    tabs: Array,
    scrollClass: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      currentTab: 0,
    };
  },
  methods: {
    onClick(index) {
      this.currentTab = index;
      this.$emit('change', index);
    },
  },
};
</script>

<style></style>
