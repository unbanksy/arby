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


        <ul class="list-group">
          <li class="list-group-item">
            <a href="https://sushiswap.vision/pair/0x088ee5007c98a9677165d78dd2109ae4a3d04d0c" target="_blank">
              YFI-ETH SLP
            </a> price: {{yfiEthPrice}}
          </li>

          <li class="list-group-item">
            <a href="https://sushiswap.vision/pair/0x9ac60b8b33092c2c0b4ca5af0dec2bcb84657e12" target="_blank">
              WOOFY-ETH SLP
            </a>
            price: {{woofyEthPrice}}
          </li>

          <li class="list-group-item">
            Assuming a {{inputETH}} ETH purchase, you'll get {{outputETH}} ETH via ETH -> YFI -> WOOFY -> ETH trade.
          </li>

          <li class="list-group-item  list-group-item-info fw-bold">
            This translates to an arbitrage opportunity of {{outputETH - inputETH}} ETH.
          </li>

          <li class="list-group-item">
            NOTE: This ignores slippage and gas tx costs. That (most probably) erases the arb opportunity.
          </li>
        </ul>

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
        yfiEthPrice: null,
        woofyEthPrice: null,
        outputETH: null
      };
    },

    mounted() {
      (async () => {
        const provider = new ethers.providers.Web3Provider(window['ethereum']);

        // YFI-ETH:https://sushiswap.vision/pair/0x088ee5007c98a9677165d78dd2109ae4a3d04d0c
        // https://etherscan.io/address/0x088ee5007c98a9677165d78dd2109ae4a3d04d0c
        const yfiETH      = new ethers.Contract("0x088ee5007c98a9677165d78dd2109ae4a3d04d0c", YfiETH, provider);
        const reserves    = await yfiETH.getReserves();
        this.yfiEthPrice  = reserves[0] / reserves[1];

        // WOOFY-ETH: https://sushiswap.vision/pair/0x9ac60b8b33092c2c0b4ca5af0dec2bcb84657e12
        // https://etherscan.io/address/0x9ac60b8b33092c2c0b4ca5af0dec2bcb84657e12
        const woofyETH     = new ethers.Contract("0x9ac60b8b33092c2c0b4ca5af0dec2bcb84657e12", WoofyETH, provider);
        const wethr        = await woofyETH.getReserves();
        this.woofyEthPrice = wethr[0] / (wethr[1] * Math.pow(10, 6));

        console.log("this.yfiEthPrice = ", this.yfiEthPrice)
        console.log("this.woofyEthPrice = ", this.woofyEthPrice)

        const yfi    = this.inputETH * this.yfiEthPrice;
        const woofy  = yfi * Math.pow(10, 6);
        const newEth = this.woofyEthPrice * woofy;
        this.outputETH = newEth;

      })();

    }





  };

</script>
