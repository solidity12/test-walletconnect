<template>
  <div>
    <button @click="connectWallet">Connect Wallet</button>
  </div>
  <div v-if="account != null">account: {{ account }}</div>
</template>

<script>
import { ref } from "vue";
import UniversalProvider from "@walletconnect/universal-provider";
// import { WalletConnectModal } from "@walletconnect/modal";
import { ethers } from "ethers";

export default {
  setup() {
    const account = ref(null);

    const connectWallet = async () => {
      try {
        const projectId = "2a2a5978a58aad734d13a2d194ec469a";

        // const modal = new WalletConnectModal({ projectId });

        const provider = await UniversalProvider.init({
          logger: "info",
          relayUrl: "wss://relay.walletconnect.com",
          projectId: projectId,
          metadata: {
            name: "WalletConnect Universal Provider",
            description: "WalletConnect Universal Provider Demo",
            url: "https://walletconnect.com/",
            icons: ["https://avatars.githubusercontent.com/u/37784886"],
          },
        });

        console.log("session", provider.session);

        await provider.connect({
          namespaces: {
            logger: "info",
            eip155: {
              methods: ["eth_sendTransaction", "eth_signTypedData"],
              chains: ["eip155:1"],
              events: ["chainChanged", "accountsChanged"],
              rpcMap: {
                1: `https://rpc.walletconnect.com/v1/?chainId=eip155:1&projectId=${projectId}`,
              },
            },
          },
        });

        console.log("hihi");

        const ethersWeb3Provider = new ethers.BrowserProvider(provider);
        const signer = await ethersWeb3Provider.getSigner();
        const account = signer.address;

        console.log("account", account);

        provider.on("display_uri", (uri) => {
          console.log("display_uri", uri);
          // modal.closeModal();
          // modal.openModal({ uri });
        });
      } catch (error) {
        console.error("Failed to connect wallet", error);
      }
    };

    return {
      connectWallet,
      account,
    };
  },
};
</script>

<style scoped>
button {
  padding: 10px 20px;
  font-size: 16px;
}
</style>
