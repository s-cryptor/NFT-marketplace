<template>
  <v-row justify="center" align="center">
    <h1 v-if="!loaded || !nfts.length">No NFTs listed</h1>
    <div>
      <div>
        <div v-for="nft in nfts" :key="nft.tokenId">
          <img :src="nft.image" />
          <div>
            <p>Price - {{ nft.price }} Eth</p>
          </div>
        </div>
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
  name: 'DashboardPage',

  data() {
    return {
      nfts: [],
      loaded: false,
    }
  },

  created() {
    this.loadNFTs()
  },

  methods: {
    async loadNFTs() {
      const web3Modal = new Web3Modal({
        network: 'mainnet',
        cacheProvider: true,
      })
      const connection = await web3Modal.connect()
      const provider = new ethers.providers.Web3Provider(connection)
      const signer = provider.getSigner()

      const contract = new ethers.Contract(
        marketplaceAddress,
        NFTMarketplace.abi,
        signer
      )
      const data = await contract.fetchItemsListed()

      const items = await Promise.all(
        data.map(async (i) => {
          const tokenUri = await contract.tokenURI(i.tokenId)
          const meta = await axios.get(tokenUri)
          const price = ethers.utils.formatUnits(i.price.toString(), 'ether')
          const item = {
            price,
            tokenId: i.tokenId.toNumber(),
            seller: i.seller,
            owner: i.owner,
            image: meta.data.image,
          }
          return item
        })
      )

      this.nfts = items
      this.loaded = true
    },
  },
}
</script>
