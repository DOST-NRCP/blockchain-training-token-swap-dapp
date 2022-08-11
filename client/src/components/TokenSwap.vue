<template>
  <div>
    <button v-if="accounts.length === 0" @click="connectToMetamask()"> Connect to Metamask </button>
    <button v-else @click="disconnect()">Disconnect</button>
    <div>Account: {{ accounts[0] }} </div>
    <div>TokenA Balance: {{ tokenABalance }} TKA </div>
    <div>TokenB Balance: {{ tokenBBalance }} TKB </div>

    <div class="container">
      <h1 class="title">Token Swap</h1>

      <div class="form-container">
        <form @submit.prevent="createSwapOffer()">
          <div class="input-group">
            <label>Swap Type</label>
            <select v-model="swapForm.swapType" class="input-field">
              <option :value="0">Token A for Token B</option>
              <option :value="1">Token B for Token A</option>
            </select>
          </div>
          <div class="input-group">
            <label>Amount From</label>
            <input v-model.number="swapForm.amountFrom" class="input-field" />
          </div>
          <div class="input-group">
            <label>Amount To</label>
            <input v-model.number="swapForm.amountTo" class="input-field" />
          </div>
          <button type="submit">Create Swap</button>
        </form>
      </div>

      <div v-for="(swapOffer, index) in swapOffers" :key="index" class="swap-card">
          <div>Swap offer by: {{ swapOffer.creator }}</div>
          <div v-if="swapOffer.swapType === 0">
              {{ swapOffer.fromTokens }} Token A 
              for {{ swapOffer.toTokens }} Token B
          </div>

          <div v-else-if="swapOffer.swapType === 1">
              {{ swapOffer.fromTokens }} Token B 
              for {{ swapOffer.toTokens }} Token A
          </div>
          <button v-if="!swapOffer.isSwapped" @click="fulfillSwap(swapOffer)">Swap</button>
      </div>
    </div>
  </div>  
</template>
<script>
import { ethers, utils } from 'ethers'
import { tokenAAddress, 
  tokenBAddress, 
  tokenSwapAddress,
  tokenAAbi, 
  tokenBAbi, 
  tokenSwapAbi
  } from "../utils/constants"

export default {
  data() {
    return {
      errorMessage: '',
      message: '',
      swapForm: {
         swapType: null,
         amountFrom: 0,
         amountTo: 0
      },
      provider: null,
      accounts: [],
      tokenSwap: null,
      tokenA: null,
      tokenB: null,
      tokenABalance: 0,
      tokenBBalance: 0,
      swapOffers: []
    }
  },
  methods: {
    async connectToMetamask() {
      this.provider = new ethers.providers.Web3Provider(window.ethereum)
      this.accounts = await this.provider.send('eth_requestAccounts', [])

      this.createContractInstances()
    },
    disconnect() {
      this.provider = null
      this.accounts = []

      this.tokenSwap = null
      this.tokenA = null
      this.tokenB = null

      this.tokenABalance = 0
      this.tokenBBalance = 0

      this.swapOffers = []
    },
    createContractInstances() {
      this.tokenSwap = new ethers.Contract(tokenSwapAddress, tokenSwapAbi)
      this.tokenSwap  = this.tokenSwap.connect(this.provider)

      this.tokenA = new ethers.Contract(tokenAAddress, tokenAAbi)
      this.tokenA  = this.tokenA.connect(this.provider)

      this.tokenB = new ethers.Contract(tokenBAddress, tokenBAbi)
      this.tokenB  = this.tokenB.connect(this.provider)

      this.getTokenBalances()
      this.getSwapsList()
    },
    async getTokenBalances() {
      const _tokenABalance = await this.tokenA.balanceOf(this.accounts[0])
      this.tokenABalance = utils.formatUnits(_tokenABalance, 18)

      const _tokenBBalance = await this.tokenB.balanceOf(this.accounts[0])
      this.tokenBBalance = utils.formatUnits(_tokenBBalance, 18)
    },
    async createSwapOffer() {
      const signer = this.provider.getSigner()
      const tokenSwapWithSigner =  this.tokenSwap.connect(signer)

      const amountFrom = utils.parseUnits(
        String(this.swapForm.amountFrom), 18)

      const amountTo = utils.parseUnits(
        String(this.swapForm.amountTo), 18)

      let tokenFrom
      if(this.swapForm.swapType === 0) {
        tokenFrom = this.tokenA
      } else {
        tokenFrom = this.tokenB
      }

      const tokenFromWithSigner = tokenFrom.connect(signer)

      try {

        const approveTx = await tokenFromWithSigner.approve(
          this.tokenSwap.address,
          amountFrom
        )
        await approveTx.wait()

        const transaction = await tokenSwapWithSigner.createSwap(
          this.swapForm.swapType,
          amountFrom,
          amountTo
        )

        await transaction.wait()

        this.errorMessage = ''
        this.getTokenBalances()
        this.getSwapsList()

      } catch (error) {
        this.errorMessage = error.data.data.reason || error.data.message
      }
    },
    async getSwapsList(){
      const swapsLength = await this.tokenSwap.getSwapsLength()
      this.swapOffers = []
      for(let i = 0; i < swapsLength; i++ ) {
        const _swap = await this.tokenSwap.swaps(i)
        const swap = {
          swapType: _swap.swapType,
          creator: _swap.creator,
          swappedBy: _swap.swappedBy,
          amountFrom: _swap.amountFrom,
          amountTo: _swap.amountTo,
          isSwapped: _swap.isSwapped,
        }        
        swap.swapIndex = i
        swap.fromTokens = utils.formatUnits(swap.amountFrom, 18)
        swap.toTokens = utils.formatUnits(swap.amountTo, 18)
        this.swapOffers.push(swap)
      }
    },
    async fulfillSwap(swapOffer) {
      const signer = this.provider.getSigner()
      const tokenSwapWithSigner = this.tokenSwap.connect(signer)
      
      let tokenTo
      if(swapOffer.swapType === 0) {
        tokenTo = this.tokenA
      } else {
        tokenTo = this.tokenB
      }

      const tokenToWithSigner = tokenTo.connect(signer)

      try {

        const approveTx = await tokenToWithSigner.approve(
          this.tokenSwap.address,
          swapOffer.amountTo
        )
        await approveTx.wait()

        const transaction = await tokenSwapWithSigner.swap(swapOffer.swapIndex)
        await transaction.wait()

        this.errorMessage = ''
        this.getTokenBalances()
        this.getSwapsList()

      } catch (error) {
        this.errorMessage = error.data.data.reason || error.data.message
      }
    } 
  },
}
</script>
<style scoped>
button {
  color: white;
  background-color: green;
  padding: 10px 20px;
  margin: 5px;
  border: none;
  border-radius:  5px;
}
button:active {
  background-color: darkgreen;
}

.card-text {
  color: #fff;
}

.input-group {
  width: 100%;
}

label {
  color: white;
  text-align: left;
  margin-right:  5px;
}

.input-field {
  margin: 10px 0px;
  padding: 5px 3px;
  width: 90%;
}

.swap-card {
  color:white;
  border: #f5f5f5 solid 1px;
  margin: 5px 0px;
  padding: 5px 0px;
}
</style>