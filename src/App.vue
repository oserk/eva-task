<template>
  <div>
    <div v-if="isReady">
      <div style="text-align: end">
        <select v-model="requestedDate">
          <option value="7">Last 7 days</option>
          <option value="14">Last 14 days</option>
          <option value="30">Last 30 days</option>
          <option value="60">Last 60 days</option>
        </select>
      </div>
      <DailySalesChart
        :chartData="chartData"
        @set-selected-date="setSkuListTable"
      />
      <div style="display: flex; justify-content: flex-end; margin: 10px 0">
        <div style="display: flex; align-items: center; gap: 3px">
          <button
            :disabled="currentPage === 1 || !showTable"
            @click="currentPage--"
          >
            &lt;
          </button>
          <div>{{ currentPage }}</div>
          <button
            :disabled="!showTable || !hasTableRows"
            @click="currentPage++"
          >
            &gt;
          </button>
        </div>
      </div>
    </div>
    <SkuListTable
      v-if="showTable && hasTableRows"
      :tableData="skuListData"
      :selectedDates="selectedDates"
      :skuRefundRates="skuRefundRates"
    />
    <PageLoader v-if="!isReady || isLoading" />
  </div>
</template>

<script>
import axios from 'axios';
import DailySalesChart from './components/DailySalesChart.vue';
import PageLoader from './components/PageLoader.vue';
import SkuListTable from './components/SkuListTable.vue';

export default {
  name: 'App',
  components: {
    DailySalesChart,
    PageLoader,
    SkuListTable,
  },
  data() {
    return {
      isReady: false,
      isLoading: true,
      showTable: false,
      token: '',
      base_url: 'https://iapitest.eva.guru',
      config: {},
      email: 'homework@eva.guru',
      userInfo: {},
      selectedStore: {},
      chartData: [],
      skuListData: {},
      skuRefundRates: [],
      selectedDates: [],
      requestedDate: '30',
      currentPage: 1,
    };
  },
  async created() {
    await this.setToken();
    this.setConfig();
    await this.setUserInfo();
    this.setSelectedStore();
    await this.setChartData();
    this.isReady = true;
    this.isLoading = false;
  },
  methods: {
    async setToken() {
      const params = {
        Email: this.email,
        Password: 'Homeworkeva1**',
        GrantType: 'password',
        Scope: 'amazon_data',
        ClientId: 'C0001',
        ClientSecret: 'SECRET0001',
        RedirectUri: 'https://api.eva.guru',
      };

      try {
        await axios
          .post(`${this.base_url}/oauth/token`, params)
          .then((response) => {
            this.token = response.data.Data.AccessToken;
          });
      } catch (error) {
        console.log(error);
      }
    },
    setConfig() {
      this.config = {
        headers: { Authorization: `Bearer ${this.token}` },
      };
    },
    async setUserInfo() {
      const params = {
        email: this.email,
      };

      try {
        await axios
          .post(`${this.base_url}/user/user-information`, params, this.config)
          .then((response) => {
            this.userInfo = response.data.Data;
          });
      } catch (error) {
        console.log(error);
      }
    },
    setSelectedStore() {
      this.selectedStore = this.userInfo.user.store[0];
    },
    async setChartData() {
      const params = {
        marketplace: this.selectedStore.marketplaceName,
        sellerId: this.selectedStore.storeId,
        requestStatus: 0,
        day: 0,
        excludeYoYData: true,
      };

      try {
        await axios
          .post(
            `${this.base_url}/data/daily-sales-overview`,
            params,
            this.config
          )
          .then((response) => {
            this.chartData = response.data.Data;
          });
      } catch (error) {
        console.log(error);
      }
    },
    async setSkuListTable(value) {
      this.isLoading = true;
      this.selectedDates = [...value];

      const params = {
        marketplace: this.selectedStore.marketplaceName,
        sellerId: this.selectedStore.storeId,
        salesDate: value[0],
        pageSize: 30,
        pageNumber: this.currentPage,
        isDaysCompare: 0,
      };

      if (value.length === 2) {
        params.salesDate2 = value[1];
        params.isDaysCompare = 1;
      }

      try {
        await axios
          .post(
            `${this.base_url}/data/daily-sales-sku-list`,
            params,
            this.config
          )
          .then(async (response) => {
            this.skuListData = response.data.Data;
            if (value.length === 1) {
              await this.setSkuRefundRate();
            }
            this.showTable = true;
            this.isLoading = false;
          });
      } catch (error) {
        console.log(error);
      }
    },
    async setSkuRefundRate() {
      const params = {
        marketplace: this.selectedStore.marketplaceName,
        sellerId: this.selectedStore.storeId,
        skuList: this.skuListData.item.skuList,
        requestedDay: this.requestedDate,
      };

      try {
        await axios
          .post(
            `${this.base_url}/data/get-sku-refund-rate`,
            params,
            this.config
          )
          .then((response) => {
            this.skuRefundRates =
              response.data.Data.length > 1
                ? response.data.Data.map((refundItem) => refundItem.refundRate)
                : [];
          });
      } catch (error) {
        console.log(error);
      }
    },
  },
  computed: {
    hasTableRows() {
      return Boolean(this.skuListData.item.skuList.length);
    },
  },
  watch: {
    requestedDate() {
      this.setSkuListTable(this.selectedDates);
    },
    currentPage() {
      this.setSkuListTable(this.selectedDates);
    },
  },
};
</script>

<style></style>
