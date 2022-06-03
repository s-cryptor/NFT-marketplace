<template>
  <v-row justify="center" align="center">
    <div>
      <div>
        <input v-model="formInput.price" placeholder="Asset Price in Eth" />
        {{ formInput.image }}
        <img width="350" :src="formInput.image" />
        <button @click="listNFTForSale">List NFT</button>
      </div>
    </div>
  </v-row>
</template>

<script>
import { ethers } from 'ethers'
import axios from 'axios'
import Web3Modal from 'web3modal'
import { marketplaceAddress } from '../config'
import NFTMarketplace from '../artifacts/contracts/NFTMarketplace.sol/NFTMarketplace.json'

export default {
  name: 'ResellNFTPage',

  data() {
    return {
      formInput: {
        price: '',
        image: '',
      },
    }
  },

  watch: {
    'this.$router.query.id'() {
      this.fetchNFT()
    },
  },

  created() {
    this.fetchNFT()
  },

  methods: {
    async fetchNFT() {
      if (!this.$router.query.tokenURI) return
      const meta = await axios.get(this.$router.query.tokenURI)
      this.formInput.image = meta.data.image
    },
    async listNFTForSale() {
      if (!this.formInput.price) return
      const web3Modal = new Web3Modal()
      const connection = await web3Modal.connect()
      const provider = new ethers.providers.Web3Provider(connection)
      const signer = provider.getSigner()

      const priceFormatted = ethers.utils.parseUnits(
        this.formInput.price,
        'ether'
      )
      const contract = new ethers.Contract(
        marketplaceAddress,
        NFTMarketplace.abi,
        signer
      )
      let listingPrice = await contract.getListingPrice()

      listingPrice = listingPrice.toString()
      const transaction = await contract.resellToken(
        this.$router.query.id,
        priceFormatted,
        { value: listingPrice }
      )
      await transaction.wait()

      this.$router.push('/')
    },
  },
}
</script>
