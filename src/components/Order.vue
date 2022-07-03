<template>
  <div>
    <v-btn @click="openOpder">Open Order</v-btn>
    <v-btn v-if="isOpderOpen">Close Order +10% profit</v-btn>
    <v-btn v-if="isOpderOpen">Close Order -10% loss</v-btn>
  </div>
</template>

<script>
import TonWeb from "tonweb";

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

const tonweb = new TonWeb(new TonWeb.HttpProvider('https://testnet.toncenter.com/api/v2/jsonRPC', {apiKey: '03f370faf426369d0f92d1272cf29590104d28c88d9471bef7f77179d163f27a'}));

export default {
  data: () => ({
    isOpderOpen: false,
    balance: 0
  }),
  methods: {
    async openOpder () {
      let publicKeyService = TonWeb.utils.hexToBytes(
        "4b2b915e9c810854c6eab3000802576a2cebbb933b11f87d3b1b9453371ab12a"
      );
      let secretKeyService = TonWeb.utils.hexToBytes(
        "5cf314617ce72ed073ed7fed95599a1531d972fe7d486b1acfbc39f52673f57e4b2b915e9c810854c6eab3000802576a2cebbb933b11f87d3b1b9453371ab12a"
      );

      let keyPairService = {
        publicKey: publicKeyService,
        secretKey: secretKeyService,
      };

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

      let id = new TonWeb.utils.BN(localStorage.getItem('channelID'))
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

      const channelB = tonweb.payments.createChannel({
        ...channelConfig,
        isA: false,
        myKeyPair: keyPairService,
        hisPublicKey: keyPairClient.publicKey,
      });

      // fee state
      const channelState1 = {
        balanceA: TonWeb.utils.toNano("0.99"),
        balanceB: TonWeb.utils.toNano("1.01"),
        seqnoA: new TonWeb.utils.BN(1),
        seqnoB: new TonWeb.utils.BN(0),
      };

      // A signs this state and send signed state to B (e.g. via websocket)
      const signatureA1 = await channelB.signState(channelState1);
      await sleep(5000)
      // B checks that the state is changed according to the rules, signs this state, send signed state to A (e.g. via websocket)

      if (!(await channelA.verifyState(channelState1, signatureA1))) {
        throw new Error("Invalid A signature");
      }

      await sleep(5000)

      const signatureB1 = await channelA.signState(channelState1);

      await sleep(5000)

      console.log(await channelB.getChannelState());
      const data = await channelB.getData();
      console.log("balanceA = ", data.balanceA.toString());
      console.log("balanceB = ", data.balanceB.toString());

      console.log(await channelA.getChannelState());
      const data2 = await channelA.getData();
      console.log("balanceA = ", data2.balanceA.toString());
      console.log("balanceB = ", data2.balanceB.toString());

    }
  },
};
</script>
