<template>
  <v-row justify="center" align="center">
    <h1 v-if="!loaded && nft.length">No items in marketplace</h1>
    <div>
      <div v-for="nft in nfts" :key="nft.tokenId">
        <img :src="nft.image" />
        <div>
          <p>{{ nft.name }}</p>
          <div>
            <p>{{ nft.description }}</p>
          </div>
        </div>
        <div>
          <p>{{ nft.price }} ETH</p>
          <button @click="buyNFT(nft)">Buy</button>
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
  name: 'IndexPage',

  data() {
    return {
      nfts: [],
      loaded: false,
    }
  },

  created: {
    async loadNFTs() {
      /* create a generic provider and query for unsold market items */
      const provider = new ethers.providers.JsonRpcProvider()
      const contract = new ethers.Contract(
        marketplaceAddress,
        NFTMarketplace.abi,
        provider
      )
      const data = await contract.fetchMarketItems()

      /*
       *  map over items returned from smart contract and format
       *  them as well as fetch their token metadata
       */
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
            name: meta.data.name,
            description: meta.data.description,
          }
          return item
        })
      )

      this.nfts = items
      this.loaded = true
    },
    async buyNFT(nft) {
      /* needs the user to sign the transaction, so will use Web3Provider and sign it */
      const web3Modal = new Web3Modal()
      const connection = await web3Modal.connect()
      const provider = new ethers.providers.Web3Provider(connection)
      const signer = provider.getSigner()
      const contract = new ethers.Contract(
        marketplaceAddress,
        NFTMarketplace.abi,
        signer
      )

      /* user will be prompted to pay the asking proces to complete the transaction */
      const price = ethers.utils.parseUnits(nft.price.toString(), 'ether')
      const transaction = await contract.createMarketSale(nft.tokenId, {
        value: price,
      })
      await transaction.wait()
      this.loadNFTs()
    },
  },
}
</script>
if (loadingState === 'loaded' && !nfts.length) return () return ( ) }
