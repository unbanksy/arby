<template>
  <div id="app" class="container">
    <div class="row">
      <div class="col-12 text-center">

        <h1>Is there a WOOFY-ETH arbitrage?</h1>
        <p>
          This uses
          <a href="https://sushiswap.vision/pair/0x088ee5007c98a9677165d78dd2109ae4a3d04d0c" target="_blank">
            YFI-ETH SLP
          </a>
          and
          <a href="https://sushiswap.vision/pair/0x9ac60b8b33092c2c0b4ca5af0dec2bcb84657e12" target="_blank">
            WOOFY-ETH SLP
          </a>
          to determine an arbitrage opportunity.

        </p>

        <p>
          <a href="https://sushiswap.vision/pair/0x088ee5007c98a9677165d78dd2109ae4a3d04d0c" target="_blank">
            YFI-ETH SLP
          </a> price: {{ethToYfiPrice}}
        </p>
        <p>
          <a href="https://sushiswap.vision/pair/0x9ac60b8b33092c2c0b4ca5af0dec2bcb84657e12" target="_blank">
            WOOFY-ETH SLP
          </a>
          price: {{woofyToEthPrice}}
        </p>

        <div class="row align-items-center mb-2">
          <div class="col-sm-6 offset-sm-3">
            <div class="input-group mb-3">
              <span class="input-group-text" id="basic-addon3">Trade Size (in ETH)</span>
              <input type="text" class="form-control" v-model="inputETH"  />
            </div>

          </div>
        </div>


        <div class="row">
          <div class="col-sm-6">
            <ul class="list-group">
              <li class="list-group-item fw-bold">
                ETH -> YFI -> WOOFY -> ETH trade
              </li>

              <li class="list-group-item">
                You'll get {{tradeYfiToWoofy}} ETH
              </li>

              <li v-bind:class="['list-group-item', 'fw-bold', tradeYfiToWoofy - inputETH > 0 ? 'list-group-item-success' : 'list-group-item-danger']">
                Arbitrage opportunity: {{tradeYfiToWoofy - inputETH}} ETH
              </li>

              <li class="list-group-item">
                NOTE: This ignores slippage and gas tx costs. That (most probably) erases the arb opportunity.
              </li>
            </ul>
          </div>

          <div class="col-sm-6">
            <ul class="list-group">
              <li class="list-group-item fw-bold">
                ETH -> WOOFY -> YFI -> ETH trade
              </li>

              <li class="list-group-item">
                You'll get {{tradeWoofyToYfi}} ETH
              </li>


              <li v-bind:class="['list-group-item', 'fw-bold', tradeWoofyToYfi - inputETH > 0 ? 'list-group-item-success' : 'list-group-item-danger']">
                Arbitrage opportunity: {{tradeWoofyToYfi - inputETH}} ETH
              </li>

              <li class="list-group-item">
                NOTE: This ignores slippage and gas tx costs. That (most probably) erases the arb opportunity.
              </li>
            </ul>
          </div>
        </div>




      </div>
    </div>
  </div>

</template>


<script>
  import { ethers } from 'ethers';
  import { abi as YfiETH } from './abi/YFI_ETH.json';
  import { abi as WoofyETH } from './abi/WOOFY_ETH.json';

  export default {
    data() {
      return {
        inputETH: 1,
        tradeWoofyToYfi: null,
        tradeYfiToWoofy: null,
      };
    },

    mounted() {
      (async () => {
        const provider = new ethers.providers.Web3Provider(window['ethereum']);

        // YFI-ETH:https://sushiswap.vision/pair/0x088ee5007c98a9677165d78dd2109ae4a3d04d0c
        // https://etherscan.io/address/0x088ee5007c98a9677165d78dd2109ae4a3d04d0c
        const yfiETH      = new ethers.Contract("0x088ee5007c98a9677165d78dd2109ae4a3d04d0c", YfiETH, provider);
        const reserves    = await yfiETH.getReserves();
        this.ethToYfiPrice  = reserves[0] / reserves[1];

        // WOOFY-ETH: https://sushiswap.vision/pair/0x9ac60b8b33092c2c0b4ca5af0dec2bcb84657e12
        // https://etherscan.io/address/0x9ac60b8b33092c2c0b4ca5af0dec2bcb84657e12
        const woofyETH     = new ethers.Contract("0x9ac60b8b33092c2c0b4ca5af0dec2bcb84657e12", WoofyETH, provider);
        const wethr        = await woofyETH.getReserves();
        this.woofyToEthPrice = wethr[0] / (wethr[1] * Math.pow(10, 6));

        this.tradeYfiToWoofy = this.inputETH * this.ethToYfiPrice * Math.pow(10, 6) * this.woofyToEthPrice;

        this.tradeWoofyToYfi = (this.inputETH * (1 / this.woofyToEthPrice) / Math.pow(10, 6)) * (1 / this.ethToYfiPrice);



      })();

    },

    methods: {
      setEthSize(evt) {
        console.log("evt.target.value = ", evt.target.value)
        this.ethSize = evt.target.value;
      }
    }





  };

</script>
