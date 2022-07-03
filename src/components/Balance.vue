<template>
  <span style="padding-left:10px;">Balance in Service: {{balance}}</span>
</template>

<script>
import TonWeb from "tonweb";

const tonweb = new TonWeb(new TonWeb.HttpProvider('https://testnet.toncenter.com/api/v2/jsonRPC', {apiKey: '03f370faf426369d0f92d1272cf29590104d28c88d9471bef7f77179d163f27a'}));

export default {
  data: () => ({
    account: "",
    balance: 0
  }),

  methods: {
    async update() {
      let publicKeyService = TonWeb.utils.hexToBytes(
        "4b2b915e9c810854c6eab3000802576a2cebbb933b11f87d3b1b9453371ab12a"
      );
      let secretKeyService = TonWeb.utils.hexToBytes(
        "5cf314617ce72ed073ed7fed95599a1531d972fe7d486b1acfbc39f52673f57e4b2b915e9c810854c6eab3000802576a2cebbb933b11f87d3b1b9453371ab12a"
      );

      let keyPairClient = {
        publicKey: TonWeb.utils.hexToBytes(localStorage.getItem('publicKey')),
        secretKey: TonWeb.utils.hexToBytes(localStorage.getItem('secretKey')),
      };

      const walletService = tonweb.wallet.create({
        publicKey: publicKeyService,
      });
      const walletAddressService = await walletService.getAddress(); // address of this wallet in blockchain
      console.log("walletAddressA = ", walletAddressService.toString(true, true, true));

      const walletClient = tonweb.wallet.create({
        publicKey: keyPairClient.publicKey,
      });
      const walletAddressClient = await walletClient.getAddress();
      console.log("walletAddressClient = ", walletAddressClient.toString(true, true, true));

      const channelInitState = {
        balanceA: TonWeb.utils.toNano("1"), // A's initial balance in Toncoins. Next A will need to make a top-up for this amount
        balanceB: TonWeb.utils.toNano("1"), // B's initial balance in Toncoins. Next B will need to make a top-up for this amount
        seqnoA: new TonWeb.utils.BN(0), // initially 0
        seqnoB: new TonWeb.utils.BN(0), // initially 0
      };

      const channelConfig = {
        channelId: new TonWeb.utils.BN(localStorage.getItem('channelID')), // Channel ID, for each new channel there must be a new ID
        addressA: walletAddressService, // A's funds will be withdrawn to this wallet address after the channel is closed
        addressB: walletAddressClient, // B's funds will be withdrawn to this wallet address after the channel is closed
        initBalanceA: channelInitState.balanceA,
        initBalanceB: channelInitState.balanceB,
      };

      const channelA = tonweb.payments.createChannel({
        ...channelConfig,
        isA: true,
        myKeyPair: keyPairClient,
        hisPublicKey: publicKeyService,
      });

      console.log(await channelA.getChannelState());
      const data = await channelA.getData();
      console.log("balanceA = ", data.balanceA.toString());
      console.log("balanceB = ", data.balanceB.toString());

      this.balance = TonWeb.utils.fromNano(data.balanceB.toString())
    }
  },

  mounted() {
    this.update()
  },
};
</script>
