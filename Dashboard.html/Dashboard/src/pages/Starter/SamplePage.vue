<template>
  <div class="row">
    <div class="col-12">
      <h3 class="card-title">Facebook API</h3>
      <card>
        <div class="row">
          <div class="col-lg-8 col-md-12 col-12">
            <div class="row">
              <div class="col-lg-4 col-md-12 col-12">
                <div class="d-flex">
                  <div class>
                    <label for>Choose Account</label>
                  </div>
                  <div class="ml-auto">
                    <input
                      type="checkbox"
                      id="accounts_checkbox"
                      v-model="selectAccounts"
                      @change="selectAllAccounts()"
                      inline
                    />
                    <label class="ml-2" for="accounts_checkbox">All</label>
                  </div>
                </div>

                <base-input>
                  <el-select
                    multiple
                    class="select-info"
                    size="large"
                    v-model="accounts_select.multiple"
                    collapse-tags
                    filterable
                    @change="checkIfAllAccountsSelected()"
                    @visible-change="onSelectAccountOnFocus($event)"
                  >
                    <el-option
                      v-for="option in accounts_select.options"
                      v-bind:class="{'selected':option.selected}"
                      :value="option.value"
                      :label="option.label"
                      v-bind:key="option.id"
                    ></el-option>
                  </el-select>
                </base-input>
              </div>

              <div class="col-lg-8 col-md-12 col-12">
                <div class="d-flex">
                  <div class>
                    <label for>Choose Campaign</label>
                  </div>
                  <div class="ml-auto">
                    <input
                      type="checkbox"
                      id="accounts_checkbox"
                      v-model="selectCampaigns"
                      @change="selectAllCampaigns()"
                    />
                    <label class="ml-2" for="campaign_checkbox">All</label>
                  </div>
                </div>
                <base-input>
                  <el-select
                    multiple
                    class="select-info"
                    size="large"
                    v-model="campaigns_select.multiple"
                    collapse-tags
                    filterable
                    @change="checkIfAllCampaignsSelected()"
                  >
                    <el-option
                      v-for="option in campaigns_select.options"
                      class="select-info"
                      :value="option.value"
                      :label="option.label"
                      :key="option.id"
                    ></el-option>
                  </el-select>
                </base-input>
              </div>
            </div>
          </div>

          <div class="col-lg-3 col-md-12 col-12">
            <div class="row">
              <div class="col-md-6 col-12">
                <label for>Start Date</label>
                <base-input>
                  <el-date-picker
                    type="date"
                    placeholder="Select Date"
                    id="start_date"
                    name="start_date"
                    v-model="start_date"
                    :picker-options="start_date_picker_options"
                  ></el-date-picker>
                </base-input>
              </div>

              <div class="col-md-6 col-12">
                <label for>End Date</label>
                <base-input>
                  <el-date-picker
                    type="date"
                    placeholder="Select Date"
                    id="end_date"
                    name="end_date"
                    v-model="end_date"
                    :picker-options="end_date_picker_options"
                  ></el-date-picker>
                </base-input>
              </div>
            </div>
          </div>

          <div class="col-lg-1 col-md-12 col-12">
            <label for></label>
            <base-button
              type="default"
              class="btn-block"
              @click.prevent="fetchAllAccountsAndCampaignsMetrics()"
            >
              <i class="tim-icons icon-refresh-02"></i>
            </base-button>
          </div>
        </div>
      </card>
    </div>

    <!-- Metrics Card -->
    <div class="col-12">
      <div class="row">
        <!--  -->
        <div class="col-lg-4 col-md-4 col-12"></div>
      </div>
    </div>
    <!-- Metrics Card -->

    <!-- Stats Cards -->
    <div class="col-lg-3 col-md-6 col-6" v-for="card in statsCards" :key="card.subTitle">
      <stats-card
        :title="card.value"
        :sub-title="card.subTitle"
        :type="card.type"
        :icon="card.icon"
      ></stats-card>
    </div>

    <div class="col-md-6 col-12 ml-auto">
      <card class="card-chart" no-footer-line>
        <template slot="header">
          <h5 class="card-category">TOTAL</h5>
          <h3 class="card-title">Clicks and Impressions</h3>
        </template>
        <div class="chart-area">
          <line-chart
            style="height: 100%"
            :chart-data="lineChartForTotalImpressionsAndClicks.chartData"
            :gradient-colors="lineChartForTotalImpressionsAndClicks.gradientColors"
            :gradient-stops="lineChartForTotalImpressionsAndClicks.gradientStops"
            :extra-options="lineChartForTotalImpressionsAndClicks.extraOptions"
          ></line-chart>
        </div>
      </card>
    </div>

    <div class="col-md-6 col-12 ml-auto">
      <card class="card-chart" no-footer-line>
        <template slot="header">
          <h5 class="card-category">TOTAL</h5>
          <h3 class="card-title">Clicks and Inline Link Clicks</h3>
        </template>
        <div class="chart-area">
          <bar-chart
            :chart-data="barChartForTotalClicksAndInlineLinkClicks.chartData"
            :extra-options="barChartForTotalClicksAndInlineLinkClicks.extraOptions"
            style="height: 100%;"
          ></bar-chart>
        </div>
      </card>
    </div>

    <div class="col-12">
      <card type="chart">
        <div class="row pl-2 pr-2 mb-2">
          <div class="col-md-6 col-12"></div>

          <div class="col-md-6 col-12">
            <div class="btn-group btn-group-toggle float-right" data-toggle="buttons">
              <label
                v-for="(option, index) in bigLineChartCategories"
                :key="option.name"
                class="btn btn-sm btn-default btn-simple"
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
import { TimeSelect, DatePicker, Select, Option, Button } from "element-ui";
import LineChart from "@/components/Charts/LineChart";
import * as chartConfigs from "@/components/Charts/config";
import config from "@/config";
import * as firebase from "firebase";
import { firebaseConfig } from "./config";
import BarChart from "src/components/Charts/BarChart";
import StatsCard from "src/components/Cards/StatsCard";

let bigChartData = [];
let bigChartLabels = [];
let accounts = [];
let campaigns = [];

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

//initialize firebase
firebase.initializeApp(firebaseConfig);
let insightsRef = firebase.database().ref("facebook-api");

export default {
  name: "starter-page",
  components: {
    LineChart,
    BarChart,
    [DatePicker.name]: DatePicker,
    [Option.name]: Option,
    [Select.name]: Select,
    Button,
    StatsCard
  },
  data() {
    return {
      select_account_has_focus: false,
      selectedAccounts: {},
      selectedCampaigns: {},
      selectAccounts: false,
      selectCampaigns: false,
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
        extraOptions: chartConfigs.lineChartOptionsForTotalImpressionsAndCLicks,
        gradientColors: config.colors.primaryGradient,
        gradientStops: [1, 0.4, 0],
        categories: []
      },
      start_date: "",
      end_date: "",
      start_date_picker_options: {
        disabledDate(time) {
          return time.getTime() > Date.now();
        },
        shortcuts: [
          {
            text: "Yesterday",
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24);
              picker.$emit("pick", date);
            }
          },
          {
            text: "A week ago",
            onClick(picker) {
              const date = new Date();
              date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", date);
            }
          },
          {
            text: "First day of the month",
            onClick(picker) {
              const date = new Date(),
                y = date.getFullYear(),
                m = date.getMonth();
              var firstDay = new Date(y, m, 1);
              date.setTime(firstDay);
              picker.$emit("pick", date);
            }
          }
        ]
      },
      end_date_picker_options: {
        disabledDate(time) {
          return time.getTime() > Date.now();
        },
        shortcuts: [
          {
            text: "Today",
            onClick(picker) {
              picker.$emit("pick", new Date());
            }
          }
        ]
      },
      datePickerOptions: {
        minimum_date: "",
        maximum_date: ""
      },
      accounts_select: {
        simple: "",
        multiple: "ARS",
        options: []
      },
      campaigns_select: {
        simple: "",
        multiple: "ARS",
        options: []
      },
      modelValidations: {
        accounts_select: {
          required: true
        },
        campaigns_select: {
          required: true
        },
        start_date: {
          required: true
        },
        end_date: {
          required: true
        }
      },
      scorecard_statistics: {
        total_clicks: 0,
        total_impressions: 0,
        total_reach: 0
      },
      statsCards: [
        {
          value: "0",
          subTitle: "Impressions",
          type: "info",
          icon: "tim-icons icon-refresh-01",
          footer: '<i class="tim-icons icon-refresh-01"></i> Update Now'
        },
        {
          value: "0",
          subTitle: "Clicks",
          type: "info",
          icon: "tim-icons icon-tap-02",
          footer: '<i class="tim-icons icon-sound-wave"></i></i> Last Research'
        },
        {
          value: "0",
          subTitle: "Reach",
          type: "info",
          icon: "tim-icons icon-refresh-01",
          footer: '<i class="tim-icons icon-trophy"></i> Customer feedback'
        },
        {
          value: "0",
          subTitle: "Click Rate",
          type: "info",
          icon: "tim-icons icon-refresh-01",
          footer: '<i class="tim-icons icon-trophy"></i> Customer feedback'
        }
      ],
      lineChartForTotalImpressionsAndClicks: {
        chartData: {
          labels: ["JUL", "AUG", "SEP", "OCT", "NOV", "DEC"],
          datasets: [
            {
              label: "Impressions",
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
              pointRadius: 4,
              data: [80, 100, 70, 80, 120, 80]
            },
            {
              label: "Clicks",
              fill: true,
              borderColor: config.colors.teal,
              borderWidth: 2,
              borderDash: [],
              borderDashOffset: 0.0,
              pointBackgroundColor: config.colors.teal,
              pointBorderColor: "rgba(255,255,255,0)",
              pointHoverBackgroundColor: config.colors.teal,
              pointBorderWidth: 20,
              pointHoverRadius: 4,
              pointHoverBorderWidth: 15,
              pointRadius: 4,
              data: [20, 60, 40, 30, 100, 40]
            }
          ]
        },
        extraOptions: chartConfigs.lineChartOptionsForTotalImpressionsAndCLicks,
        gradientColors: config.colors.primaryGradient,
        gradientStops: [1, 0.4, 0]
      },
      barChartForTotalClicksAndInlineLinkClicks: {
        chartData: {
          labels: ["JUL", "AUG", "SEP", "OCT", "NOV", "DEC"],
          datasets: [
            {
              label: "Data",
              fill: true,
              backgroundColor: config.colors.primary,
              hoverBackgroundColor: config.colors.primary,
              borderColor: config.colors.primary,
              borderWidth: 2,
              borderDash: [],
              borderDashOffset: 0.0,
              data: [80, 100, 70, 80, 120, 80]
            },
            {
              label: "Data",
              fill: true,
              backgroundColor: config.colors.teal,
              hoverBackgroundColor: config.colors.teal,
              borderColor: config.colors.teal,
              borderWidth: 2,
              borderDash: [],
              borderDashOffset: 0.0,
              data: [60, 110, 90, 70, 90, 100]
            }
          ]
        },
        extraOptions: chartConfigs.barChartOptionsGradient
      }
    };
  },
  methods: {
    //set chart from data
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

    //fetching All Available Accounts and Campaigns from FB API
    fetchAllAccountsAndCampaignsFromFacebookApi() {
      insightsRef.once("value").then(snapshot => {
        snapshot.forEach(childSnapshot => {
          //get the random generated id
          var key = childSnapshot.key;

          //get the data inside object
          var values = snapshot.child(childSnapshot.key).val();

          var accounts = [];
          var campaigns = [];

          //loop to get all the accounts and campaigns
          values.data.forEach(key => {
            accounts.push({
              id: key.account_id,
              label: key.account_name,
              value: key.account_id,
              selected: false
            });
            campaigns.push({
              id: key.campaign_id,
              label: key.campaign_name,
              value: key.campaign_id,
              selected: false
            });
          });

          accounts.sort((a, b) => (a.color > b.color ? 1 : -1));
          campaigns.sort((a, b) => (a.color > b.color ? 1 : -1));

          this.accounts_select.options = this.removeDuplicateId(accounts, "id");
          this.campaigns_select.options = this.removeDuplicateId(
            campaigns,
            "id"
          );
        });
      });
    },
    removeDuplicateId(myArr, prop) {
      return myArr.filter((obj, pos, arr) => {
        return arr.map(mapObj => mapObj[prop]).indexOf(obj[prop]) === pos;
      });
    },
    selectAllAccounts() {
      //method to select all accounts
      let selected_option = this.accounts_select.options;
      let multiple_value = this.accounts_select.multiple;

      //condition to check if the select all accounts checkbox is checked
      if (this.selectAccounts == true) {
        selected_option.forEach(option => {
          //check if the account is already selected
          if (!multiple_value.includes(option.value.toString())) {
            multiple_value.push(option.value.toString());
          }
        });
        this.fetchAllCampaignsFromSelectedAccounts(
          this.accounts_select.multiple
        );
      } else {
        multiple_value.splice(0, multiple_value.length);
      }
    },
    selectAllCampaigns() {
      let selected_option = this.campaigns_select.options;
      let multiple_value = this.campaigns_select.multiple;
      if (this.selectCampaigns == true) {
        selected_option.forEach(option => {
          if (!multiple_value.includes(option.value.toString())) {
            multiple_value.push(option.value.toString());
          }
        });
      } else {
        multiple_value.splice(0, multiple_value.length);
      }
    },
    checkIfAllAccountsSelected() {
      let multiple_value = this.accounts_select.multiple;
      let all_options = this.accounts_select.options;

      if (multiple_value.length == all_options.length) {
        this.selectAccounts = true;
      } else {
        this.selectAccounts = false;
      }
    },
    checkIfAllCampaignsSelected() {
      let multiple_value = this.campaigns_select.multiple;
      let all_options = this.campaigns_select.options;

      if (multiple_value.length == all_options.length) {
        this.selectCampaigns = true;
      } else {
        this.selectCampaigns = false;
      }
    },
    onSelectAccountOnFocus(elementClosed) {
      if (elementClosed == false) {
        this.fetchAllCampaignsFromSelectedAccounts(
          this.accounts_select.multiple
        );
      }
    },
    fetchAllCampaignsFromSelectedAccounts(selectedAccountsId) {
      insightsRef.once("value").then(snapshot => {
        snapshot.forEach(childSnapshot => {
          //get the random generated id
          var key = childSnapshot.key;

          //get the data inside object
          var values = snapshot.child(childSnapshot.key).val();

          //loop to get all the
          var campaigns = [];
          this.campaigns_select.options = [];
          this.campaigns_select.multiple = [];
          this.selectCampaigns = false;

          //loop to get all the campaigns of all selected accounts
          selectedAccountsId.forEach(account_id => {
            values.data.forEach((key, value) => {
              if (account_id == key.account_id) {
                campaigns.push({
                  id: key.campaign_id,
                  label: key.campaign_name,
                  value: key.campaign_id,
                  account_id: key.account_id,
                  account_name: key.account_name,
                  selected: false
                });
              }
            });

            this.campaigns_select.options = this.removeDuplicateId(
              campaigns,
              "id"
            );
          });
        });
      });
    },

    //Fetching Metrics from selected accounts and campaings between start date and end date
    fetchAllAccountsAndCampaignsMetrics() {
      //form validation
      if (this.accounts_select.multiple.length <= 0) {
        alert("No Accounts Selected");
      } else if (this.campaigns_select.multiple.length <= 0) {
        alert("No Campaigns Selected");
      } else if (!this.start_date) {
        alert("No Start Date Selected");
      } else if (!this.end_date) {
        alert("No End Date Selected");
      } else {
        console.log("Fetching Metrics");

        insightsRef.once("value").then(snapshot => {
          snapshot.forEach(childSnapshot => {
            //get the random generated id
            var key = childSnapshot.key;

            //get the data inside object
            var values = snapshot.child(childSnapshot.key).val();

            //array for each dates data
            var metrics = [];

            var date_start = new Date(this.start_date);
            var date_end = new Date(this.end_date);

            var cpc_value = 0;
            var cpm_value = 0;
            var cpp_value = 0;
            var ctr_value = 0;
            var total_clicks = 0;
            var total_impressions = 0;
            var total_reach = 0;

            this.campaigns_select.multiple.forEach(campaign_id => {
              values.data.forEach((key, value) => {
                var campaign_date_start = new Date(
                  new Date(key.date_start).toDateString()
                );
                var campaign_date_stop = new Date(
                  new Date(key.date_stop).toDateString()
                );

                if (campaign_id == key.campaign_id) {
                  if (
                    campaign_date_start >= date_start &&
                    campaign_date_stop <= date_end
                  ) {
                    if (key.cpc !== undefined) cpc_value = parseFloat(key.cpc);
                    else {
                      cpc_value = parseFloat(0);
                    }

                    if (key.cpm !== undefined) cpm_value = parseFloat(key.cpm);
                    else {
                      cpm_value = parseFloat(0);
                    }

                    if (key.cpp !== undefined) cpp_value = parseFloat(key.cpp);
                    else {
                      cpp_value = parseFloat(0);
                    }

                    if (key.ctr !== undefined) ctr_value = parseFloat(key.ctr);
                    else {
                      ctr_value = parseFloat(0);
                    }

                    if (key.clicks !== undefined)
                      total_clicks += parseFloat(key.clicks);
                    else {
                      total_clicks += parseFloat(0);
                    }

                    if (key.impressions !== undefined)
                      total_impressions += parseFloat(key.impressions);
                    else {
                      total_impressions += parseFloat(0);
                    }

                    if (key.reach !== undefined)
                      total_reach += parseFloat(key.reach);
                    else {
                      total_reach += parseFloat(0);
                    }

                    //condition to check whether the date start of key is already in metrics array
                    if (
                      metrics.filter(
                        e => e.date_start == key.date_start.toString()
                      ).length > 0
                    ) {
                      //loop to add the cpp, cpm, cpp, ctr data to metrics array with the same date
                      for (var i = 0; i < metrics.length; i++) {
                        if (key.date_start == metrics[i].date_start) {
                          metrics[i].data.cpc += cpc_value;
                          metrics[i].data.cpm += cpm_value;
                          metrics[i].data.cpp += cpp_value;
                          metrics[i].data.ctr += ctr_value;
                        }
                      }
                    } else {
                      var properties = {
                        date_start: key.date_start.toString(),
                        data: {
                          cpc: cpc_value,
                          cpm: cpm_value,
                          cpp: cpp_value,
                          ctr: ctr_value
                        }
                      };
                      metrics.push(properties);
                    }
                  }
                }
              });
            });
            /* add to chart data */

            //arrays to save total metrics for each
            var cpc = [];
            var cpm = [];
            var cpp = [];
            var ctr = [];
            var label = [];
            bigChartData = [];
            bigChartLabels = [];

            //Assign the scorecard sum
            this.statsCards[1]["value"] = total_clicks.toLocaleString();
            this.statsCards[0]["value"] = total_impressions.toLocaleString();
            this.statsCards[2]["value"] = total_reach.toLocaleString();
            this.statsCards[3]["value"] = (
              total_clicks / total_impressions
            ).toLocaleString();
            //save total metrics to each
            metrics.forEach(key => {
              cpc.push(key.data.cpc);
              cpm.push(key.data.cpm);
              cpp.push(key.data.cpp);
              ctr.push(key.data.ctr);
              label.push(key.date_start.toString());
            });

            //push data to chart
            bigChartData.push(cpc);
            bigChartData.push(cpm);
            bigChartData.push(cpp);
            bigChartData.push(ctr);
            bigChartLabels = label;

            //draw chart
            this.initBigChart(0);
          });
        });
      }
    },
    initializeLineChartForTotalImpressionsAndClicksData() {}
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
    //draw chart when page is loaded
    this.fetchAllAccountsAndCampaignsFromFacebookApi();
    insightsRef.once("value").then(snapshot => {
      snapshot.forEach(childSnapshot => {
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
        var accounts = [];
        var campaigns = [];
        var total_clicks = 0;
        var total_impressions = 0;
        var total_reach = 0;

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

          if (key.clicks !== undefined) total_clicks += parseFloat(key.clicks);
          else {
            total_clicks += parseFloat(0);
          }

          if (key.impressions !== undefined)
            total_impressions += parseFloat(key.impressions);
          else {
            total_impressions += parseFloat(0);
          }

          if (key.reach !== undefined) total_reach += parseFloat(key.reach);
          else {
            total_reach += parseFloat(0);
          }

          if (key.clicks !== undefined) total_clicks += parseFloat(key.clicks);
          else {
            total_clicks += parseFloat(0);
          }

          if (key.impressions !== undefined)
            total_impressions += parseFloat(key.impressions);
          else {
            total_impressions += parseFloat(0);
          }

          if (key.reach !== undefined) total_reach += parseFloat(key.reach);
          else {
            total_reach += parseFloat(0);
          }

          var date_start = key.date_start.toString();
          var date_stop = key.date_stop.toString();
          label.push(key.date_start.toString());
        });

        //Assign the scorecard sum
        this.statsCards[0]["value"] = total_impressions.toLocaleString();
        this.statsCards[1]["value"] = total_clicks.toLocaleString();
        this.statsCards[2]["value"] = total_reach.toLocaleString();
        this.statsCards[3]["value"] =
          (total_clicks / total_impressions).toLocaleString() + "%";

        bigChartData.push(cpc);
        bigChartData.push(cpm);
        bigChartData.push(cpp);
        bigChartData.push(ctr);
        bigChartLabels = label;
      });
      this.initBigChart(0);
    });
  }
};
</script>
<style></style>