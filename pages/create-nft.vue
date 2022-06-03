<template>
  <v-row justify="center" align="center">
    <div>
      <div>
        <input v-model="formInput.name" placeholder="Asset Name" />
        <textarea
          v-model="formInput.description"
          placeholder="Asset Description"
        />
        <input v-model="formInput.price" placeholder="Asset Price in Eth" />
        <input type="file" name="Asset" @change="onChange" />
        {{ fileUrl }}
        <img width="350" :src="fileUrl" />
        <button @click="listNFTForSale">Create NFT</button>
      </div>
    </div>
  </v-row>
</template>

<script>
import { ethers } from 'ethers'
import { create as ipfsHttpClient } from 'ipfs-http-client'
import Web3Modal from 'web3modal'
import { marketplaceAddress } from '../config'
import NFTMarketplace from '../artifacts/contracts/NFTMarketplace.sol/NFTMarketplace.json'

const client = ipfsHttpClient('https://ipfs.infura.io:5001/api/v0')

export default {
  name: 'CreateNFTPage',

  data() {
    return {
      fileUrl: '',
      formInput: {
        price: '',
        name: '',
        description: '',
      },
    }
  },

  methods: {
    async onChange(e) {
      /* upload image to IPFS */
      const file = e.target.files[0]
      try {
        const added = await client.add(file, {
          progress: (prog) => console.log(`received: ${prog}`),
        })
        this.fileUrl = `https://ipfs.infura.io/ipfs/${added.path}`
      } catch (error) {
        console.log('Error uploading file: ', error)
      }
    },
    async uploadToIPFS() {
      const { name, description, price } = this.formInput
      if (!name || !description || !price || !this.fileUrl) return
      /* first, upload metadata to IPFS */
      const data = JSON.stringify({
        name,
        description,
        image: this.fileUrl,
      })
      try {
        const added = await client.add(data)
        const url = `https://ipfs.infura.io/ipfs/${added.path}`
        /* after metadata is uploaded to IPFS, return the URL to use it in the transaction */
        return url
      } catch (error) {
        console.log('Error uploading file: ', error)
      }
    },
    async listNFTForSale() {
      const url = await this.uploadToIPFS()
      const web3Modal = new Web3Modal()
      const connection = await web3Modal.connect()
      const provider = new ethers.providers.Web3Provider(connection)
      const signer = provider.getSigner()

      /* create the NFT */
      const price = ethers.utils.parseUnits(this.formInput.price, 'ether')
      const contract = new ethers.Contract(
        marketplaceAddress,
        NFTMarketplace.abi,
        signer
      )
      let listingPrice = await contract.getListingPrice()
      listingPrice = listingPrice.toString()
      const transaction = await contract.createToken(url, price, {
        value: listingPrice,
      })
      await transaction.wait()

      this.$router.push('/')
    },
  },
}
</script>
