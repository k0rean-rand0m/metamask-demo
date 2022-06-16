<template>
  <div class="hello">
    <h3>MetaMask + Vue.js demo</h3>
    <p>
      <a href="https://metamask.io/download/" target="_blank">Install Metamask</a><br>
      Claim free Eth + DAI for the tests <a href="https://faucet.paradigm.xyz/" target="_blank">here</a><br>
      and switch MetaMask to Rinkeby Network...<br>
      ...or you'll be forced ha-ha
    </p>


    <a href="#" @click="connect">{{address === undefined ? "Connect" : address}}</a><br><br>
    <!-- Balances are stored and processed in 1/(10^18) of currency since Ethereum doesn't support float numbers -->
    <!-- Here we use web3.utils.fromWei to format Wei balance (1/(10^18) of currency) to pretty numbers  -->
    <div>ETH balance: {{web3.utils.fromWei(balance)}}</div>
    <div>DAI balance: {{web3.utils.fromWei(daiBalance)}}</div>


    <h3>Sending DAI</h3><br>
    Send DAI to: <input v-model="daiRecipient" style="width: 30vw" placeholder="Send DAI to..."><br>
    Amount of DAI to send: <input v-model="daiAmountToSend" style="width: 30vw" placeholder="Amount to send"><br><br>
    <button @click="transferDai">Send DAI</button><br><br>


    <!-- Here we'll display all DAI transfer transactions during the session -->
    <!-- With a nice link to block explorer, where we'll be able to discover the receipt -->
    Transactions:
    <div v-for="tx in sessionTxs" v-bind:key="tx.transactionHash">
      <a :href="`https://rinkeby.etherscan.io/tx/${tx.transactionHash}`"
         target="_blank">
        {{tx.transactionHash}}
      </a>
    </div>
  </div>
</template>

<script>
import Web3 from "web3"
export default {
  name: 'Demo',
  props: {
    msg: String
  },

  data() {
    return {
      web3: undefined,

      address: undefined,
      balance: "",

      daiContract: undefined,
      daiBalance: "",
      daiRecipient: "0x3A205ECf286bBe11460638aCe47D501A53fB91C0",
      daiAmountToSend: "1",

      sessionTxs: []
    }
  },

  beforeMount() {
    // Checking MetaMask browser extension is installed
    // MetaMask extension is a gateway to blockchain from a browser window
    if (typeof window.ethereum !== 'undefined') {
      // Initializing a an object of Web3 which will allow us to work with blockchain. It requires a "provider".
      // window.ethereum is a "provider" which is provided by MetaMask extension. It allows us to "connect to blockchain"
      this.web3 = new Web3(window.ethereum);
    }
  },

  async mounted() {
    // "Connecting" to a DAI smart-contract (like usual API, but in a blockchain)
    this.daiContract = new this.web3.eth.Contract(
        // Interface (like available HTTP-endpoints in usual APIs). We determine it so Web3 will be able to create valid transactions (a.k.a requests)
        [{"inputs":[{"internalType":"uint256","name":"chainId_","type":"uint256"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"src","type":"address"},{"indexed":true,"internalType":"address","name":"guy","type":"address"},{"indexed":false,"internalType":"uint256","name":"wad","type":"uint256"}],"name":"Approval","type":"event"},{"anonymous":true,"inputs":[{"indexed":true,"internalType":"bytes4","name":"sig","type":"bytes4"},{"indexed":true,"internalType":"address","name":"usr","type":"address"},{"indexed":true,"internalType":"bytes32","name":"arg1","type":"bytes32"},{"indexed":true,"internalType":"bytes32","name":"arg2","type":"bytes32"},{"indexed":false,"internalType":"bytes","name":"data","type":"bytes"}],"name":"LogNote","type":"event"},{"anonymous":false,"inputs":[{"indexed":true,"internalType":"address","name":"src","type":"address"},{"indexed":true,"internalType":"address","name":"dst","type":"address"},{"indexed":false,"internalType":"uint256","name":"wad","type":"uint256"}],"name":"Transfer","type":"event"},{"constant":true,"inputs":[],"name":"DOMAIN_SEPARATOR","outputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"PERMIT_TYPEHASH","outputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"internalType":"address","name":"","type":"address"},{"internalType":"address","name":"","type":"address"}],"name":"allowance","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"usr","type":"address"},{"internalType":"uint256","name":"wad","type":"uint256"}],"name":"approve","outputs":[{"internalType":"bool","name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"balanceOf","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"usr","type":"address"},{"internalType":"uint256","name":"wad","type":"uint256"}],"name":"burn","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"decimals","outputs":[{"internalType":"uint8","name":"","type":"uint8"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"guy","type":"address"}],"name":"deny","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"usr","type":"address"},{"internalType":"uint256","name":"wad","type":"uint256"}],"name":"mint","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"src","type":"address"},{"internalType":"address","name":"dst","type":"address"},{"internalType":"uint256","name":"wad","type":"uint256"}],"name":"move","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"name","outputs":[{"internalType":"string","name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"nonces","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"holder","type":"address"},{"internalType":"address","name":"spender","type":"address"},{"internalType":"uint256","name":"nonce","type":"uint256"},{"internalType":"uint256","name":"expiry","type":"uint256"},{"internalType":"bool","name":"allowed","type":"bool"},{"internalType":"uint8","name":"v","type":"uint8"},{"internalType":"bytes32","name":"r","type":"bytes32"},{"internalType":"bytes32","name":"s","type":"bytes32"}],"name":"permit","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"usr","type":"address"},{"internalType":"uint256","name":"wad","type":"uint256"}],"name":"pull","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"usr","type":"address"},{"internalType":"uint256","name":"wad","type":"uint256"}],"name":"push","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"guy","type":"address"}],"name":"rely","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"symbol","outputs":[{"internalType":"string","name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"dst","type":"address"},{"internalType":"uint256","name":"wad","type":"uint256"}],"name":"transfer","outputs":[{"internalType":"bool","name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"internalType":"address","name":"src","type":"address"},{"internalType":"address","name":"dst","type":"address"},{"internalType":"uint256","name":"wad","type":"uint256"}],"name":"transferFrom","outputs":[{"internalType":"bool","name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"version","outputs":[{"internalType":"string","name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"wards","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}],
        // Blockchain address of DAI smart-contract (it's like a base URL in usual APIs)
        "0x6A9865aDE2B6207dAAC49f8bCba9705dEB0B0e6D"
    )
  },

  methods:{
    async connect() {
      // Requesting MetaMask browser extension to get user's accounts
      const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
      // Get a primary account
      // It will be in a HEX format with leading "0x" following with 40 HEX-symbols ( [0-9, A-F, a-f] )
      this.address = accounts[0];

      // Switching the network to Rinkeby testnet. We don't wanna operate with real funds now :)
      await window.ethereum.request({
        method: 'wallet_switchEthereumChain',
        params: [{ chainId: '0x4' }],  // The id of Rinkeby Chain is 4. Should be passed here in HEX
      });

      // Get Eth balance
      await this.getBalanceEth()
      // Get DAI balance
      await this.getDaiBalance()
    },

    async getBalanceEth() {
      // Eth is a base currency of Ethereum, so there's no smart-contract behind
      // Eth is used to pay gas fees (transactions (requests) commission),
      //    but might be used as money, it have utility, why not? ;)
      this.balance = await this.web3.eth.getBalance(this.address)
    },

    async getDaiBalance() {
      // DAI, like every cryptocurrency is a smart-contract
      // You can imagine it like a bank which runs its own bank accounts with balances and its own currency
      // So to get users' balance we need to request their "API"

      // Here we use initialized in mounted() DAI smart-contract (blockchain DAI API endpoint) to request users balance
      // It's available through Web3.eth.Contract.methods.balanceOf(address), where balanceOf is the name of contract function (like an API endpoint)
      // This "endpoint" requires an address of account for which we're requesting a balance as a parameter
      // Generally speaking this exact method is a standard for a token which DAI is, but smart-contract methods might be implemented in any possible ways, like an any "classic" API might be
      // .call() is used to call a method. It's like GET in usual APIs. It will just read from the blockchain and won't change anything
      // .call()-ing is free for users
      // { from: this.account } - I use it to determine who's calling the method. Maybe it'll work without this argument, but it's a nice practice to determine the caller
      this.daiBalance = await this.daiContract.methods.balanceOf(this.address).call({ from: this.address })
    },

    async transferDai() {
      // Preparing amount to send
      // As it was mentioned earlier, balances are stored and processed in 1/(10^18) of currency since Ethereum doesn't support float numbers
      //    so we need to multiply users input by 10^18. It's easier to use Web3.utils.toWei(AMOUNT-IN-ETH, 'ether') for that
      const amount = this.web3.utils.toWei(this.daiAmountToSend, 'ether');

      // Transfers DAI token by requesting its contract
      // All the same as it is in this.getDaiBalance(), but...
      // We use here transfer(recipient, amount) "endpoint" to transfer DAI from one account in another
      // Keep in mind that DAI balances are stored not in wallet, but in a blockchain,
      //    so we request "DAI API" to decrease tokens amount at senders account and increase it at recipients
      // Also here .send() is used which might be understood as POST request
      // It will perform actions in blockchain (changing balances in this case), so it will cost user gas (transaction (request) commission)
      // .call() won't work here and passing { from: address } to .send() is necessary
      // Also we pass { value: 0 } along with "from". That's not necessary for this method (transfer),
      //    but in general that is used to send some additional Eth with transaction.
      //    Sometimes methods accepts that, but this one don't. Anyway, I pass { value: "0" } so no Eth will be sent.
      const tx = await this.daiContract.methods.transfer(this.daiRecipient, amount).send({ from: this.address, value: "0" })

      // Transaction is a way to change blockchain state, but imagine it just as usual request
      // It have some interesting data, such Transaction Hash, Used gas, Gas Price and etc
      // Why won't to check it out?
      console.log(tx)

      // Also let's save TXs during the session to display them
      this.sessionTxs.push(tx)
      console.log(this.sessionTxs)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
