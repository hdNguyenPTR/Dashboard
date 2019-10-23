<template>
  <div class="row">
    <div class="col-12">
      <h3 class="card-title">Facebook API</h3>

      <div class="row">
        <div class="col-md-8">
          <card>
            <div class="row">
              <div class="col-6">
                <base-input>
                  <el-select
                    multiple
                    class="select-info"
                    size="large"
                    v-model="accounts_select.multiple"
                    collapse-tags
                    placeholder="Select Account"
                  >
                    <el-option
                      v-for="option in accounts_select.options"
                      class="select-info"
                      :value="option.value"
                      :label="option.label"
                      :key="option.label"
                    ></el-option>
                  </el-select>
                </base-input>
              </div>

              <div class="col-6">
                <base-input>
                  <el-select
                    multiple
                    class="select-info"
                    size="large"
                    v-model="campaigns_select.multiple"
                    collapse-tags
                    placeholder="Select Campaign"
                  >
                    <el-option
                      v-for="option in campaigns_select.options"
                      class="select-info"
                      :value="option.value"
                      :label="option.label"
                      :key="option.label"
                    ></el-option>
                  </el-select>
                </base-input>
              </div>
            </div>
          </card>
        </div>

        <div class="col-md-4">
          <card>
            <div class="row">
              <div class="col-md-6 col-12">
                <base-input>
                  <el-date-picker type="date" placeholder="Start Date" v-model="start_date"></el-date-picker>
                </base-input>
              </div>

              <div class="col-md-6 col-12">
                <base-input>
                  <el-date-picker type="date" placeholder="End Date" v-model="end_date"></el-date-picker>
                </base-input>
              </div>
            </div>
          </card>
        </div>
      </div>
    </div>

    <div class="col-12">
      <card type="chart">
        <div class="row pl-1 pr-1">
          <div class="col-md-6 col-12"></div>

          <div class="col-md-6 col-12">
            <div class="btn-group btn-group-toggle" data-toggle="buttons">
              <label
                v-for="(option, index) in bigLineChartCategories"
                :key="option.name"
                class="btn btn-sm btn-primary btn-simple"
                :class="{ active: bigLineChart.activeIndex === index }"
                :id="index"
              >
                <input
                  type="radio"
                  @click="initBigChart(index)"
                  name="options"
                  autocomplete="off"
                  :checked="bigLineChart.activeIndex === index"
                />
                <span class="d-none d-sm-block">{{ option.name }}</span>
                <span class="d-block d-sm-none">
                  <i :class="option.icon"></i>
                </span>
              </label>
            </div>
          </div>
        </div>
        <div class="chart-area">
          <line-chart
            style="height: 100%"
            ref="bigChart"
            :chart-data="bigLineChart.chartData"
            :gradient-colors="bigLineChart.gradientColors"
            :gradient-stops="bigLineChart.gradientStops"
            :extra-options="bigLineChart.extraOptions"
          ></line-chart>
        </div>
      </card>
    </div>
  </div>
</template>
<script>
import { TimeSelect, DatePicker, Select, Option } from "element-ui";
import LineChart from "@/components/Charts/LineChart";
import * as chartConfigs from "@/components/Charts/config";
import config from "@/config";
import * as firebase from "firebase";

let bigChartData = [];

let bigChartLabels = [];

let bigChartDatasetOptions = {
  fill: true,
  borderColor: config.colors.primary,
  borderWidth: 2,
  borderDash: [],
  borderDashOffset: 0.0,
  pointBackgroundColor: config.colors.primary,
  pointBorderColor: "rgba(255,255,255,0)",
  pointHoverBackgroundColor: config.colors.primary,
  pointBorderWidth: 20,
  pointHoverRadius: 4,
  pointHoverBorderWidth: 15,
  pointRadius: 4
};

var firebaseConfig = {
  apiKey: "AIzaSyA4BHgsOPKoctSbTCShfhEV3em-ACTFtW8",
  authDomain: "rising-symbol-256704.firebaseapp.com",
  databaseURL: "https://rising-symbol-256704.firebaseio.com",
  projectId: "rising-symbol-256704",
  storageBucket: "rising-symbol-256704.appspot.com",
  messagingSenderId: "724027350638",
  appId: "1:724027350638:web:edfa9d0604cc4ef921d07f",
  measurementId: "G-85YPZ0NVLW"
};

firebase.initializeApp(firebaseConfig);
      
export default {
  name: "starter-page",
  components: {
    LineChart,
    [DatePicker.name]: DatePicker,
    [Option.name]: Option,
    [Select.name]: Select
  },

  data() {
    return {
      bigLineChart: {
        activeIndex: 0,
        chartData: {
          datasets: [
            {
              ...bigChartDatasetOptions,
              data: bigChartData[0]
            }
          ],
          labels: bigChartLabels
        },
        extraOptions: chartConfigs.purpleChartOptions,
        gradientColors: config.colors.primaryGradient,
        gradientStops: [1, 0.4, 0],
        categories: []
      },
      start_date: "",
      end_date: "",
      accounts_select: {
        simple: "",
        multiple: "ARS",
        options: [{ value: "All Accounts", label: "All Accounts" }]
      },
      campaigns_select: {
        simple: "",
        multiple: "ARS",
        options: [{ value: "All Campaigns", label: "All Campaigns" }]
      }
    };
  },
  methods: {
    initBigChart(index) {
      let chartData = {
        datasets: [
          {
            ...bigChartDatasetOptions,
            data: bigChartData[index]
          }
        ],
        labels: bigChartLabels
      };
      this.$refs.bigChart.updateGradients(chartData);
      this.bigLineChart.chartData = chartData;
      this.bigLineChart.activeIndex = index;
    },
  },
  computed: {
    bigLineChartCategories() {
      return [
        { name: "CPC", icon: "", fields: "cpc" },
        { name: "CPM", icon: "", fields: "cpm" },
        { name: "CPP", icon: "", fields: "cpp" },
        { name: "CTR", icon: "", fields: "ctr" }
      ];
    }
  },
  mounted() {
    var insightsRef = firebase.database().ref("insights");

      insightsRef.once("value").then((snapshot) => {
        let list = [];

        snapshot.forEach((childSnapshot) => {
          //get the random generated id
          var key = childSnapshot.key;

          //get the data inside object
          var values = snapshot.child(childSnapshot.key).val();

          //loop to get all the
          var cpc = [];
          var cpm = [];
          var cpp = [];
          var ctr = [];
          var label = [];

          values.data.forEach((key, value) => {
            if (key.cpc !== undefined) cpc.push(parseFloat(key.cpc));
            else {
              cpc.push(parseFloat(0));
            }

            if (key.cpm !== undefined) cpm.push(parseFloat(key.cpm));
            else {
              cpm.push(parseFloat(0));
            }

            if (key.cpp !== undefined) cpp.push(parseFloat(key.cpp));
            else {
              cpp.push(parseFloat(0));
            }

            if (key.ctr !== undefined) ctr.push(parseFloat(key.ctr));
            else {
              ctr.push(parseFloat(0));
            }
            var date_start = key.date_start.toString();
            var date_stop = key.date_stop.toString();
            label.push(
              key.date_start.toString() + " - " + key.date_stop.toString()
            );
          });

          bigChartData.push(cpc);
          bigChartData.push(cpm);
          bigChartData.push(cpp);
          bigChartData.push(ctr);
          bigChartLabels = label;
        });
        this.initBigChart(0);
      });

    
  },
};
</script>
<style></style>