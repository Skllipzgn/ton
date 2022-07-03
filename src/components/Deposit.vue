<template>
  <v-btn @click="init">Deposit</v-btn>
</template>

<script>
import TonWeb from "tonweb";

const tonweb = new TonWeb(new TonWeb.HttpProvider('https://testnet.toncenter.com/api/v2/jsonRPC', {apiKey: '03f370faf426369d0f92d1272cf29590104d28c88d9471bef7f77179d163f27a'}));
const nacl = TonWeb.utils.nacl

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

export default {
  data() {
    return {
    }
  },

  methods: {
    async init() {
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

      let randomId = new TonWeb.utils.BN(TonWeb.utils.bytesToHex(nacl.randomBytes(6)), 16)
      localStorage.setItem('channelID', randomId)

      const channelConfig = {
        channelId: randomId, // Channel ID, for each new channel there must be a new ID
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

      const channelAddress = await channelA.getAddress(); // address of this payment channel smart-contract in blockchain
      console.log("channelAddress=", channelAddress.toString(true, true, true));

      const channelB = tonweb.payments.createChannel({
        ...channelConfig,
        isA: false,
        myKeyPair: keyPairService,
        hisPublicKey: keyPairClient.publicKey,
      });

      if ((await channelB.getAddress()).toString() !== channelAddress.toString()) {
        throw new Error("Channels address not same");
      }

      const fromWalletA = channelA.fromWallet({
        wallet: walletService,
        secretKey: secretKeyService,
      });

      const fromWalletB = channelB.fromWallet({
        wallet: walletClient,
        secretKey: keyPairClient.secretKey,
      });

      await fromWalletB.deploy().send(TonWeb.utils.toNano("0.05"));

      await sleep(5000)
      await fromWalletA
        .topUp({ coinsA: channelInitState.balanceA, coinsB: new TonWeb.utils.BN(0) })
        .send(channelInitState.balanceA.add(TonWeb.utils.toNano("0.05"))); // +0.05 TON to network fees

      await sleep(5000)

      await fromWalletB
        .topUp({ coinsA: new TonWeb.utils.BN(0), coinsB: channelInitState.balanceB })
        .send(channelInitState.balanceB.add(TonWeb.utils.toNano("0.05"))); // +0.05 TON to network fees

      await sleep(5000)

      await fromWalletB.init(channelInitState).send(TonWeb.utils.toNano("0.05"));
      window.location.reload
    },
  },
}
</script>