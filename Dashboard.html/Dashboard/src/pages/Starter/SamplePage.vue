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
              type="primary"
              class="btn-block"
              @click.prevent="fetchAllAccountsAndCampaignsMetrics()"
            >
              <i class="tim-icons icon-refresh-02"></i>
            </base-button>
          </div>
        </div>
      </card>
    </div>

    <div class="col-12">
      <card type="chart">
        <div class="row pl-1 pr-1">
          <div class="col-md-6 col-12"></div>

          <div class="col-md-6 col-12">
            <div class="btn-group btn-group-toggle float-right" data-toggle="buttons">
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
import { TimeSelect, DatePicker, Select, Option, Button } from "element-ui";
import LineChart from "@/components/Charts/LineChart";
import * as chartConfigs from "@/components/Charts/config";
import config from "@/config";
import * as firebase from "firebase";

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
let insightsRef = firebase.database().ref("insights");

export default {
  name: "starter-page",
  components: {
    LineChart,
    [DatePicker.name]: DatePicker,
    [Option.name]: Option,
    [Select.name]: Select,
    Button
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
        extraOptions: chartConfigs.purpleChartOptions,
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
    fetchAllAccountsAndCampaignsFromFacebookApi() {
      insightsRef.once("value").then(snapshot => {
        let list = [];

        snapshot.forEach(childSnapshot => {
          //get the random generated id
          var key = childSnapshot.key;

          //get the data inside object
          var values = snapshot.child(childSnapshot.key).val();

          //loop to get all the
          var accounts = [];
          var campaigns = [];

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
        let list = [];

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
    fetchAllAccountsAndCampaignsMetrics() {
      //form validation
      if (this.accounts_select.multiple.length <= 0) {
        console.log("No Accounts Selected");
      } else if (this.campaigns_select.multiple.length <= 0) {
        console.log("No Campaigns Selected");
      } else if (!this.start_date) {
        console.log("No Start Date Selected");
      } else if (!this.end_date) {
        console.log("No End Date Selected");
      } else {
        console.log("Fetching Metrics");

        insightsRef.once("value").then(snapshot => {
          let list = [];

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

            var metrics = [];

            var date_start = new Date(this.start_date);
            var date_end = new Date(this.end_date);

            this.campaigns_select.multiple.forEach(campaign_id => {
              values.data.forEach((key, value) => {
                if (campaign_id == key.campaign_id) {
                  var cpc_value = parseFloat(key.cpc);
                  var cpm_value = parseFloat(key.cpm);
                  var cpp_value = parseFloat(key.cpp);
                  var ctr_value = parseFloat(key.ctr);

                  //condition to check whether the date start of key is already in metrics array
                  if (
                    metrics.filter(
                      e => e.date_start == key.date_start.toString()
                    ).length > 0
                  ) {
                    //loop to add the cpp, cpm, cpp, ctr data to metrics array with the same date
                    for (var i = 0; i < metrics.length - 1; i++) {
                      if (key.date_start == metrics[i].date_start) {
                        metrics[i].data.cpc =
                          cpc_value + parseFloat(metrics[i].data.cpc);
                        metrics[i].data.cpm =
                          cpm_value + parseFloat(metrics[i].data.cpm);
                        metrics[i].data.cpp =
                          cpp_value + parseFloat(metrics[i].data.cpp);
                        metrics[i].data.ctr =
                          ctr_value + parseFloat(metrics[i].data.ctr);
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
              });
            });
          });
        });
      }
    }
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
    this.fetchAllAccountsAndCampaignsFromFacebookApi();
    insightsRef.once("value").then(snapshot => {
      let list = [];

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
          label.push(key.date_start.toString());
        });

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