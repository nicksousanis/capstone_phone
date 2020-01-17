<template>
  <view class="container">
    <text>{{ title }}</text>
    <text></text>
    <button v-if="time" v-bind:onPress="startTimer" :title="btnTitle1" />
    <button v-if="time === false" v-bind:onPress="stopTimer" :title="btnTitle3" />
    <text></text>
    <text class="text-container">{{ totalHours }}:{{ totalMinutes }}:{{ totalSeconds }}</text>
    <text></text>
    <button v-if="reset" v-bind:onPress="pauseTimer" :title="btnTitle2" />
    <button v-if="reset === false" v-bind:onPress="resumeTimer" :title="btnTitle5" />
    <text></text>
    <text>Distance: {{ displayDistance }} miles</text>
    <text></text>
    <text>{{ displayMPH }} MPH</text>
    <text></text>
    <button v-bind:onPress="clearEverything" :title="btnTitle7" />
    <text></text>
    <button v-bind:onPress="postWorkout" :title="btnTitle4" />
    <text>{{ success }}</text>
    <text></text>
    <!--     <button v-if="reset2" v-bind:onPress="getLocation" :title="btnTitle6" />
    <button v-if="reset2 === false" v-bind:onPress="stopLocation" :title="btnTitle3" /> -->
    <text></text>
  </view>
</template>

<script>
import { Constants } from "expo";
import * as Permissions from "expo-permissions";
import * as Location from "expo-location";
import * as geolib from "geolib";

import axios from "axios";

export default {
  data: function() {
    return {
      timer: null,
      testSecond: 1,
      totalHours: "00",
      totalMinutes: "00",
      totalSeconds: "00",
      allSeconds: 0,
      btnTitle1: "Start",
      btnTitle2: "Pause",
      title: "Start timer when ready",
      btnTitle3: "Stop",
      recordedTime: null,
      btnTitle4: "Post Workout",
      success: "",
      reset: true,
      btnTitle5: "Resume",
      time: true,
      errorMessage: "",
      locations: [],
      btnTitle6: "Get Location",
      timer2: null,
      reset2: true,
      totalDistance: 0,
      newDistance: 0,
      displayDistance: "0.000",
      milesPerHour: 0,
      timer3: null,
      displayMPH: "0.00",
      btnTitle7: "Clear",
      postDistance: 3.33
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
      this.allSeconds = this.allSeconds + 1;
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
      this.recordedTime = null;
      this.totalSeconds = "00";
      this.totalMinutes = "00";
      this.totalHours = "00";
      this.timer = setInterval(this.updateTime, 1000);
      this.title = "Keep running!!";
      this.time = false;
      this.displayDistance = "0.000";
      this.timer2 = setInterval(this.updateLocation, 5000);
      this.reset2 = false;
      this.timer3 = setInterval(this.updateMPH, 5000);
    },
    pauseTimer: function() {
      clearInterval(this.timer);
      this.timer = null;
      this.title = "Never quit, keep going!!";
      this.reset = false;
      clearInterval(this.timer2);
      this.timer2 = null;
      this.reset2 = true;
      clearInterval(this.timer3);
      this.timer3 = null;
    },
    resumeTimer: function() {
      this.timer = setInterval(this.updateTime, 1000);
      this.title = "Keep running!!";
      this.reset = true;
      this.timer2 = setInterval(this.updateLocation, 5000);
      this.reset2 = false;
      this.timer3 = setInterval(this.updateMPH, 5000);
    },
    stopTimer: function() {
      clearInterval(this.timer);
      this.timer = null;
      this.recordedTime = `${this.totalHours}:${this.totalMinutes}:${this.totalSeconds}`;

      this.title = "Nice work!!";
      this.time = true;
      clearInterval(this.timer2);
      this.timer2 = null;
      this.reset2 = true;
      this.reset = true;
      this.timer3 = null;
    },
    postWorkout: function() {
      var params = {
        workout_time: this.recordedTime,
        user_id: 1,
        distance: this.totalDistance.toFixed(3),
        locations: this.locations
      };
      axios.post("https://rocky-refuge-83349.herokuapp.com/api/workouts", params).then(response => {
        this.success = "Workout sucessfully recorded";
      });
    },
    getPermission: function() {
      Permissions.askAsync(Permissions.LOCATION)
        .then(status => {
          if (status !== "granted") {
            this.errorMessage = "Permission to access location was denied";
          }
        })
        .catch(err => {
          console.log(err);
        });
    },
    getLocation: function() {
      this.timer2 = setInterval(this.updateLocation, 5000);
      this.reset2 = false;
    },
    stopLocation: function() {
      clearInterval(this.timer2);
      this.timer2 = null;
      this.reset2 = true;
    },
    updateLocation: function() {
      Location.getCurrentPositionAsync({}).then(location1 => {
        this.locations.push(location1);
        // console.log(this.locations);
        if (this.locations.length > 1) {
          this.newDistance =
            geolib.getDistance(
              {
                latitude: this.locations[this.locations.length - 1].coords.latitude,
                longitude: this.locations[this.locations.length - 1].coords.longitude
              },
              {
                latitude: this.locations[this.locations.length - 2].coords.latitude,
                longitude: this.locations[this.locations.length - 2].coords.longitude
              }
            ) * 0.0006213712;
          this.totalDistance = this.totalDistance + this.newDistance;
          this.displayDistance = this.totalDistance.toFixed(3);
        }
        // console.log(this.locations);
      });
    },
    updateMPH: function() {
      this.milesPerHour = this.totalDistance / this.allSeconds;
      this.displayMPH = this.milesPerHour.toFixed(2);
    },
    clearEverything: function() {
      this.totalSeconds = "00";
      this.totalMinutes = "00";
      this.totalHours = "00";
      this.milesPerHour = 0;
      this.displayMPH = "0.00";
      this.totalDistance = 0;
      this.newDistance = 0;
      this.displayDistance = "0.000";
      clearInterval(this.timer);
      clearInterval(this.timer2);
      clearInterval(this.timer3);
      this.success = "";
      this.title = "Start timer when ready";
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
