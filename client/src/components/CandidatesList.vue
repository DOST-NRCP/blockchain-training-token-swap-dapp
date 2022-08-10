<template>
    <div>
        <button v-if="accounts.length === 0" @click="connectToMetamask()">
            Connect to  metamask
        </button>
        <button v-else @click="disconnect()">
            Disconnect
        </button>
        <div>Account:{{ accounts[0] }}</div>
        <div>Balance:{{ balance }} ETH</div>
        <div class="container">
            <h1 class="title">
                {{ message }}
            </h1>
            <div class="candidate-count">Number of candidates: {{ candidatesLength }}</div>
            <div v-if="errorMessage.length > 0" class="error-badge"> 
                Error: {{ errorMessage }}
            </div>
            <div class="candidate-container">    
                <div class="candidate-card" v-for="(candidate, index) in candidates" :key="index">
                    <h3>Candidate no. {{ index + 1 }}</h3>
                    <img :src="candidate.image" alt="Avatar" height="100"/>
                    <div>Name: {{ candidate.name }}</div>
                    <div>Votes: {{ candidate.voteCount }}</div>
                    <button @click="addVote(index)">Vote</button>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { ethers, utils } from "ethers"
import { contractAddress, contractABI } from "../utils/constants"

export default {
    data() {
        return {
            provider: null,
            accounts: [],
            balance: 0,
            contract: null,
            candidatesLength: 0,
            candidates: [],
            candidate: { 
                name: "Pacman", 
                voteCount: 10 
            },
            message: "Candidates List",
            errorMessage: ''
        }
    },
    methods: {
        async connectToMetamask() {
            this.provider = new ethers.providers.Web3Provider(window.ethereum)
            this.accounts = await this.provider.send('eth_requestAccounts', [])
            this.getBalance()
            this.createContractInstance()
        },
        async getBalance() {
            const rawBalance = await this.provider.getBalance(this.accounts[0])
            this.balance = utils.formatEther(rawBalance)
        },
        createContractInstance() {
            this.contract = new ethers.Contract(
                contractAddress, 
                contractABI
            )
            this.contract = this.contract.connect(this.provider)

            this.getCandidates()
        },
        async getCandidates() {
            this.candidatesLength = await this.contract.getCandidatesLength()
            this.candidates = []
            for (let i = 0; i < this.candidatesLength; i++) {
                const candidate = await this.contract.candidates(i)
                const _candidate = {
                    ...candidate, 
                    image: `https://avatars.dicebear.com/api/micah/${candidate.name}.svg`
                }
                this.candidates.push(_candidate)
            }

        },
        disconnect() {
            this.accounts = []
            this.balance = 0
            this.provider = null
            this.contract = null
            this.candidatesLength = 0
            this.candidates = []
        },
        async addVote(candidateIndex) {
            const signer = this.provider.getSigner()
            const contractWithSigner =  this.contract.connect(signer)

            try {
                const transaction = await contractWithSigner.vote(candidateIndex)
                await transaction.wait()

                this.errorMessage = ''
                this.getCandidates()
                
            } catch (error) {
                this.errorMessage = error.data.data.reason || error.data.message
            }
            
        }
    }
}
</script>
<style scoped>
    button {
        color: white;
        background-color: green;
        padding: 10px 20px;
        margin: 5px;
        border: none;
        border-radius: 5px;
    }

    button:active {
        background-color: darkgreen;
    }

    .large-text {
        font-size: 25px;
    }
</style>
