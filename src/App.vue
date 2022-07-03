<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        
      </div>

      <v-spacer></v-spacer>

      <CreateWallet />
      <Deposit />
      
      <Balance />
      <v-spacer></v-spacer>

      <Account />
    </v-app-bar>

    <v-main style="padding: 64px 0 0 0;">
      <v-container fluid style="width: 1200px; margin: 0 auto;">
        <v-row>
          <v-col style="padding: 0"><Chart /></v-col>
          <v-col style="padding: 0"><OrderBook /></v-col>
        </v-row>
        <v-row>
          <Order />
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import Account from "./components/Account";
import Chart from "./components/Chart";
import OrderBook from "./components/OrderBook";
import CreateWallet from "./components/CreateWallet";
import Deposit from "./components/Deposit";
import Balance from "./components/Balance";
import Order from "./components/Order";

export default {
  name: "App",

  components: {
    Account,
    Chart, OrderBook, CreateWallet,Deposit, Balance, Order
  },

  data: () => ({
    //
  }),
  mounted() {
    if(!localStorage.getItem('disable_CreateWallet')) {
      localStorage.setItem('disable_CreateWallet', false)
    }
  }
};
</script>

<style>


</style>