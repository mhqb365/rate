<template>
  <div class="container">
    <div class="row mt-4">
      <div class="col-sm-12 col-md-12">
        <button
          v-for="c in selectCoin"
          :key="c"
          type="button"
          class="btn mr-2"
          v-bind:class="[coin == c ? 'btn-primary' : 'btn-light']"
          @click="fetch(c)"
        >{{ c }}</button>
      </div>
    </div>

    <div class="row mt-4">
      <div class="col-sm-12 col-md-6 text-center">
        <h5>Giá mua</h5>
        <h5 class="border border-success bg-success text-white rounded p-4">
          <span v-if="isLoading">?</span>
          <span v-else>{{ buy }}</span>
        </h5>
      </div>

      <div class="col-sm-12 col-md-6 text-center">
        <h5>Giá bán</h5>
        <h5 class="border border-danger bg-danger text-white rounded p-4">
          <span v-if="isLoading">?</span>
          <span v-else>{{ sell }}</span>
        </h5>
      </div>
    </div>

    <div class="row mt-4">
      <div class="col-sm-12 col-md-12">
        <div class="form-group">
          <label>Máy tính</label>
          <input v-model="amount" type="text" class="form-control" />
        </div>
      </div>

      <div class="col-sm-12 col-md-6 text-center">
        <div class="form-group">
          <label>Mua với</label>
          <h5 v-if="isLoading">?</h5>
          <h5 v-else>{{ (amount * buy).toLocaleString() }} vnđ</h5>
        </div>
      </div>

      <div class="col-sm-12 col-md-6 text-center">
        <div class="form-group">
          <label>Bán với</label>
          <h5 v-if="isLoading">?</h5>
          <h5 v-else>{{ (amount * sell).toLocaleString() }} vnđ</h5>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      isLoading: false,
      coin: "doge",
      selectCoin: ["doge", "trx"],
      buy: 0,
      sell: 0,
      amount: 0,
    };
  },
  mounted() {
    this.fetch(this.coin);
  },
  methods: {
    fetch: function (coin) {
      this.coin = coin;
      this.isLoading = !this.isLoading;
      axios({
        url: "https://api-prod.bitmoon.net/graphql",
        method: "POST",
        data: {
          query:
            "\n    query getPriceBookPaging ($page_size: Int, $page: Int, $sort_name: String, $sort_type: String, $search: String) {\n      apiPricebookPaging (page_size: $page_size, page: $page, sort_name: $sort_name, sort_type: $sort_type, search: $search){\n        total,\n        data {\n        id,\n        coin_id,\n        coin_name,\n        bid_price_usd,\n        fast_bid_price,\n        ask_price_usd,\n        bid_price_btc,\n        ask_price_btc,\n        bid_price_vnd,\n        ask_price_vnd,\n        fast_ask_price,\n        fast,\n        normal,\n        sort_no,\n        buy_profit,\n        sell_profit,\n        change_24h,\n        volume,\n        coin\n      }\n    }\n  }\n",
          variables: {
            page_size: 50,
            page: 1,
            sort_name: "volume",
            sort_type: "desc",
            search: coin,
          },
        },
      }).then((response) => {
        this.isLoading = !this.isLoading;
        // console.log(response.data.data.apiPricebookPaging.data[0]);
        let result = response.data.data.apiPricebookPaging.data[0];
        this.buy = result.fast_ask_price;
        this.sell = result.fast_bid_price;
      });
    },
  },
};
</script>

<style>
</style>