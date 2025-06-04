<template>
  <div>
    <van-field
      readonly
      label="选择日期"
      v-model="selectedDateText"
      placeholder="请选择日期"
      @click="showCalendar = true"
    />
    <van-calendar
      v-model="selectedDate"
      :min-date="minDate"
      @confirm="onConfirm"
      @cancel="showCalendar = false"
      v-if="showCalendar"
    />
  </div>
</template>

<script>
import { Toast } from "vant";

export default {
  data() {
    return {
      selectedDate: "",
      selectedDateText: "",
      showCalendar: false,
      minDate: new Date("2000-01-01"), // 今天作为最小日期
    };
  },
  methods: {
    onConfirm(date) {
      this.selectedDate = date;
      this.selectedDateText = this.formatDate(date);
      this.showCalendar = false;
    },
    formatDate(date) {
      const year = date.getFullYear();
      const month = (date.getMonth() + 1).toString().padStart(2, "0");
      const day = date.getDate().toString().padStart(2, "0");
      return `${year}-${month}-${day}`;
    },
  },
};
</script>
