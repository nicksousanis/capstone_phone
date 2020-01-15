<template>
  <view class="container">
    <text>{{ title }}</text>
    <text></text>
    <button v-bind:onPress="startTimer" :title="btnTitle1" />
    <text></text>
    <text class="text-container">{{ totalHours }}:{{ totalMinutes }}:{{ totalSeconds }}</text>
    <text></text>
    <button v-bind:onPress="pauseTimer" :title="btnTitle2" />
    <text></text>
    <text></text>
    <button v-bind:onPress="stopTimer" :title="btnTitle3" />
    <text></text>
    <text></text>
    <button v-bind:onPress="getRaces" :title="btnTitle4" />
    <text></text>
    <text>{{ testAxios }}</text>
  </view>
</template>

<script>
import { Constants, Location, Permissions } from "expo";
import axios from "axios";

export default {
  data: function() {
    return {
      timer: null,
      testSecond: 1,
      totalHours: "00",
      totalMinutes: "00",
      totalSeconds: 55,
      btnTitle1: "Start",
      btnTitle2: "Pause",
      title: "Start timer when ready",
      btnTitle3: "Stop",
      recordedTime: null,
      testAxios: [],
      btnTitle4: "Get Races"
    };
  },
  methods: {
    handleBtnClickCount1: function() {
      this.btnClickCount = this.btnClickCount.parseInt + 1;
    },
    handleBtnClickCount2: function() {
      this.btnClickCount = 0;
    },
    updateTime: function() {
      this.totalSeconds = +this.totalSeconds + 1;
      if (this.totalSeconds <= 9) {
        this.totalSeconds = `0${this.totalSeconds}`;
      }
      if (this.totalSeconds > 59) {
        this.totalMinutes = +this.totalMinutes + 1;
        this.totalSeconds = "00";
        if (+this.totalMinutes <= 9) {
          this.totalMinutes = `0${this.totalMinutes}`;
        }
      }

      if (this.totalMinutes > 59) {
        this.totalHours = this.totalMinutes + 1;
        this.totalMinutes = 0;
      }
    },
    startTimer: function() {
      this.timer = setInterval(this.updateTime, 1000);
      this.title = "Keep running!!";
    },
    pauseTimer: function() {
      clearInterval(this.timer);
      this.timer = null;
      this.title = "Never quit, keep going!!";
    },
    stopTimer: function() {
      clearInterval(this.timer);
      this.timer = null;
      this.recordedTime = this.totalSeconds;
      this.totalSeconds = "00";
      this.totalMinutes = "00";
      this.totalHours = "00";
      this.title = "Nice work!!";
    },
    getRaces: function() {
      axios.get("http://localhost:3000/api/races").then(response => {
        this.testAxios = response.data;
      });
    }
  }
};
</script>

<style>
.container {
  background-color: white;
  align-items: center;
  justify-content: center;
  flex: 1;
}
.text-color-primary {
  color: blue;
}
</style>
