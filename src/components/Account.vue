<template>
  <v-btn @click="connect" v-if="account === ''" text>
    <span class="mr-2">Connect account</span>
    <v-icon>mdi-open-in-new</v-icon>
  </v-btn>
  <span v-else>{{account}} ({{balance}})</span>
</template>

<script>
import TonWeb from "tonweb";
import axios from 'axios'

const tonweb = new TonWeb(new TonWeb.HttpProvider('https://testnet.toncenter.com/api/v2/jsonRPC', {apiKey: '03f370faf426369d0f92d1272cf29590104d28c88d9471bef7f77179d163f27a'}));

export default {
  name: "Account",

  data: () => ({
    account: "",
    balance: 0
  }),

  methods: {
    async connect() {
      const provider = window.ton;
      const accounts = await provider.send("ton_requestAccounts");
      this.account = accounts[0]
      
                  console.log(await provider.send('ton_requestWallets'));

      this.getBalance(this.account)
    },
    async getBalance(address) {
      try {
        let {data: data} = await axios.get(`https://testnet.toncenter.com/api/v2/getAddressBalance?address=${address}`)

        this.balance = tonweb.utils.fromNano(data.result)
      } catch(e) {
        console.log('failed to get balance on account')
      }
    }
  },

  mounted() {

    const onTonReady = () => {

      if (!window.tonProtocolVersion || window.tonProtocolVersion < 1) {
        alert("Please update your TON Wallet Extension");
        return;
      }

      const provider = window.ton;
      console.log("isTonWallet=", provider.isTonWallet);

      this.connect();
    };

    if (window.ton) {
      onTonReady();
    } else {
      window.addEventListener("tonready", () => onTonReady(), false);
    }
    console.log(`the component is now mounted.`);
  },
};
</script>
