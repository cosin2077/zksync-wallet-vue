<template>
  <i-modal v-model="opened" class="wrongNetworkModal" size="md">
    <template #header>Wrong network</template>
    <div>
      <div class="_padding-bottom-1">
        You are on the wrong network. Please
          <i-button
            size="sm"
            color="primary"
            :loading="switchLoading"
            @click="switchNetwork"
          >
            switch
          </i-button>
        your wallet to <b>{{ network }}</b> to continue.
      </div>
      <div class="_padding-bottom-1 text-sm">
        Some wallets may not support changing networks. If you can not change networks in your wallet you may consider
        switching to a different wallet.
      </div>
    </div>
  </i-modal>
</template>

<script lang="ts">
import { ZkConfig, ZkEthereumNetwork } from "@matterlabs/zksync-nuxt-core/types";
import { ethers } from "ethers";
import Vue from "vue";

export default Vue.extend({
  name: "WrongNetwork",
  data: () => {
    return {
      switchLoading: false,
    }
  },
  computed: {
    network(): string {
      return (this.$store.getters["zk-onboard/config"] as ZkConfig).ethereumNetwork.name;
    },
    networkInfo(): ZkEthereumNetwork {
      return (this.$store.getters["zk-onboard/config"] as ZkConfig).ethereumNetwork;
    },
    opened: {
      set(val): void {
        if (val === false) {
          this.$store.dispatch("zk-onboard/rejectNetworkChange");
        }
      },
      get(): boolean {
        return this.$store.getters["zk-onboard/wrongNetwork"];
      },
    },
  },
  methods: {
    async switchNetwork() {
      this.switchLoading = true;
      try {
        await window.ethereum.request({
          method: 'wallet_switchEthereumChain',
          params: [{ chainId: ethers.utils.hexValue(this.networkInfo.id) }], // chainId must be in hexadecimal numbers
        });
        this.$store.dispatch("zk-onboard/rejectNetworkChange");
        this.$analytics.track("change_network", { networkName: this.network });
        this.$store.dispatch("zk-provider/changeNetwork", this.network);
      } catch (err) {
        console.log(err)
      } finally {
        this.switchLoading = false;
      }
    }
  }
});
</script>

<style lang="scss">
.wrongNetworkModal .text-sm {
  font-size: 13px;
}
</style>
