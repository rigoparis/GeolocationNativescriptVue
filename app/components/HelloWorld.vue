<template>
  <Page class="page">
    <GridLayout rows="auto, auto, auto, *, auto">
          <GridLayout row="0" columns="*, *, *, *" >
              <Button text="Enable Location" col="0" textWrap="true" @tap="enableLocationTap"/>
              <Button text="Get Current Location" col="1" textWrap="true" @tap="buttonGetLocationTap"/>
              <Button text="Start Monitoring" col="2" textWrap="true" @tap="buttonStartTap"/>
              <Button text="Stop Monitoring" col="3" textWrap="true" @tap="buttonStopTap"/>
          </GridLayout>
          <GridLayout row="1" columns="*, *" >
              <Button text="Start Background thread monitoring" col="0" ios:visibility="collapsed" textWrap="true" @tap="startBackgroundTap"/>
              <Button text="Stop Background thread monitoring" col="1" ios:visibility="collapsed" textWrap="true" @tap="stopBackgroundTap"/>
          </GridLayout>
          <Label row="2" :text="watchId"></Label>
          <ListView row="3" for="loc in model.locations">
              <v-template>
                  <Label :text="`${$index}, ${loc.latitude}, ${loc.longitude}, ${loc.altitude}`" />
              </v-template>
          </ListView>
          <Button text="Clear" row="4" @tap="buttonClearTap"/>
      </GridLayout>
  </Page>
</template>

<script>
  import * as geolocation from "nativescript-geolocation";
  import * as platform from "tns-core-modules/platform";
  import { Accuracy } from "ui/enums";
  const utils = require("tns-core-modules/utils/utils");
  import * as application from "tns-core-modules/application";
  let locationService = require('./backgroundServices');

  export default {
    data () {
      return {
        model: {
          locations: []
        },
        watchId: null,
        BGids: [],
      };
    },
    methods: {
      startBackgroundTap,
      stopBackgroundTap,
      enableLocationTap,
      buttonGetLocationTap,
      buttonStartTap,
      buttonStopTap,
      buttonClearTap
    }
  };

  function startBackgroundTap() {
    if (application.android) {
      const { sdkVersion } = platform.device;
      const context = utils.ad.getApplicationContext();
      if (sdkVersion * 1 < 26) {
        const intent = new android.content.Intent(context, com.ciriscr.geotest.location.BackgroundService.class);
        console.log("startService");
        context.startService(intent);
      } else {
        const component = new android.content.ComponentName(context, com.ciriscr.geotest.location.BackgroundService26.class);
        const builder = new android.app.job.JobInfo.Builder(1, component);
        builder.setRequiredNetworkType(android.app.job.JobInfo.NETWORK_TYPE_ANY);
        builder.setPeriodic(15 * 60 * 1000);
        const jobScheduler = context.getSystemService(android.content.Context.JOB_SCHEDULER_SERVICE);
        const service = jobScheduler.schedule(builder.build());
        this.BGids.push(service);
        console.log(`Job Scheduled: ${jobScheduler.schedule(builder.build())}`);
      }
    }
      /*if (application.android) {
          let context = utils.ad.getApplicationContext();
          let intent = new android.content.Intent(context, com.nativescript.location.BackgroundService.class);
          context.startService(intent);
      }*/
  }

  function stopBackgroundTap() {
    if (application.android) {
      const { sdkVersion } = platform.device;
      const context = utils.ad.getApplicationContext();
      if (sdkVersion * 1 < 26) {
        const intent = new android.content.Intent(context, com.ciriscr.geotest.location.BackgroundService.class);
        console.log("stopService");
        context.stopService(intent);
      } else {
        if (this.BGids.length > 0) {
          const jobScheduler = context.getSystemService(android.content.Context.JOB_SCHEDULER_SERVICE);
          const service = this.BGids.pop();
          jobScheduler.cancel(service);
          console.log(`Job Canceled: ${service}`);
        }
      }
    }
      /*if (application.android) {
          let context = utils.ad.getApplicationContext();
          let intent = new android.content.Intent(context, com.nativescript.location.BackgroundService.class);
          context.stopService(intent);
      }*/
  }

  function enableLocationTap() {
      geolocation.isEnabled().then(function (isEnabled) {
        console.log(isEnabled);
          if (!isEnabled) {
              geolocation.enableLocationRequest().then(function () {
              }, function (e) {
                  console.log("Error: " + (e.message || e));
              });
          }
      }, function (e) {
          console.log("Error: " + (e.message || e));
      });
  }

  function buttonGetLocationTap() {
    const comp = this;
    geolocation.isEnabled().then((isEnabled) => {
      if (isEnabled) {
        let location = geolocation.getCurrentLocation({
            desiredAccuracy: Accuracy.high,
            maximumAge: 5000,
            timeout: 10000,
            iosAllowsBackgroundLocationUpdates: true,
            updateDistance: 0.1,
            timeout: 20000,
        })
            .then(function (loc) {
                if (!!loc) {
                    comp.model.locations.push(loc);
                }
            }, function (e) {
                console.log("Error: " + (e.message || e));
            });
      }
    })
  }

  function buttonStartTap() {
      try {
          const comp = this;
          geolocation.isEnabled().then((isEnabled) => {
            if (isEnabled) {
              comp.watchId = geolocation.watchLocation(
                  function (loc) {
                    console.log(loc);
                      if (loc) {
                          comp.model.locations.push(loc);
                      }
                  },
                  function (e) {
                      console.log("Error: " + e.message);
                  },
                  {
                    desiredAccuracy: Accuracy.high,
                    maximumAge: 5000,
                    timeout: 20000,
                    updateDistance: 1,
                    updateTime: 3000,
                    iosAllowsBackgroundLocationUpdates: true,
                    iosPausesLocationUpdatesAutomatically: false
                  });
            }
          })
      } catch (ex) {
          console.log("Error: " + ex.message);
      }
  }

  function buttonStopTap() {
      if (this.watchId != null) {
          geolocation.clearWatch(this.watchId);
          this.watchId = null;
      }
  }

  function buttonClearTap() {
      this.model.locations.splice(0, this.model.locations.length);
  }
</script>

<style scoped>
  .hello-world {
    margin: 20;
  }

  Label {
    color: red;
  }
</style>
