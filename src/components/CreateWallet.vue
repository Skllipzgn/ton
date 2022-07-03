<template>
  <v-btn @click="init" :disabled="isDisabled">Create Wallet</v-btn>
</template>

<script>
import TonWeb from "tonweb";
const tonweb = new TonWeb(new TonWeb.HttpProvider('https://testnet.toncenter.com/api/v2/jsonRPC', {apiKey: '03f370faf426369d0f92d1272cf29590104d28c88d9471bef7f77179d163f27a'}));

const nacl = TonWeb.utils.nacl
export default {
  data() {
    return {
    }
  },
  computed: {
    isDisabled() {
      return Boolean(localStorage.getItem('disable_CreateWallet') === 'true')
    }
  },

  methods: {
    async init() {
      if(Boolean(localStorage.getItem('disable_CreateWallet') === 'true')) return
      localStorage.setItem('disable_CreateWallet', true)

      let keyPair = nacl.sign.keyPair(); // create new random key pair
      let wallet = tonweb.wallet.create({ publicKey: keyPair.publicKey }); // create interface to wallet smart contract (wallet v3 by default)
      let walletAddress = await wallet.getAddress(); // address of this wallet in blockchain
      let address = walletAddress.toString(true, true, false);

      localStorage.setItem('address', address)
      localStorage.setItem('publicKey', TonWeb.utils.bytesToHex(keyPair.publicKey))
      localStorage.setItem('secretKey', TonWeb.utils.bytesToHex(keyPair.secretKey))

      console.log(`Created address. See https://testnet.tonscan.org/address/${address}`)

      await this.sendWalletFee(address); // send fee from router

      setTimeout(async() => {
        await wallet.deploy(keyPair.secretKey).send(); // deploy
        console.log(`Deploy`)

      }, 20000)
    },
    async sendWalletFee(address) {
      let publicKey = TonWeb.utils.hexToBytes("4b2b915e9c810854c6eab3000802576a2cebbb933b11f87d3b1b9453371ab12a");
      let secretKey = TonWeb.utils.hexToBytes("5cf314617ce72ed073ed7fed95599a1531d972fe7d486b1acfbc39f52673f57e4b2b915e9c810854c6eab3000802576a2cebbb933b11f87d3b1b9453371ab12a");
      const wallet = tonweb.wallet.create({
        publicKey: publicKey,
      });
      const seqno = await wallet.methods.seqno().call(); // call get-method seqno of wallet smart contract
      const transfer = wallet.methods.transfer({
        secretKey: secretKey,
        toAddress: address,
        amount: TonWeb.utils.toNano("2"), // 0.01 TON
        seqno: seqno,
        sendMode: 1,
        bounce: false,
      });
      const transferSended = await transfer.send(); // send transfer query to blockchain
      console.log(`Sending 2 TON to new address`)

    }
  },
}
</script>