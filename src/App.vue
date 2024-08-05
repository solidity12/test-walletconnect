<template>
  <div>
    <button @click="connectWallet">Connect Wallet</button>
  </div>
  <div v-if="account != null">account: {{ account }}</div>
</template>

<script>
import { ref } from "vue";
import UniversalProvider from "@walletconnect/universal-provider";
import { WalletConnectModal } from "@walletconnect/modal";
import { ethers } from "ethers";

export default {
  setup() {
    const projectId = "cde71aa9c96dc7043ab05bc0a17b0f88";
    const web3Modal = new WalletConnectModal({
      projectId,
      themeMode: "light",
      themeVariables: {
        "--wcm-accent-color": "#000000",
        "--wcm-background-color": "#000000",
      },
      explorerRecommendedWalletIds: [
        "8eefa62c2f86b4c73bd68cf5cb178e09d15cbf21399ac5aea234d2b616e9ae9d",
        "fa9c3adc4f0bbe263db1565d200f776e5da900ead0f1914e0ecbf8b313d268e9",
        "c57ca95b47569778a828d19178114f4db188b89b763c899ba0be274e97267d96",
        // "225affb176778569276e484e1b92637ad061b01e13a048b35a9d280c3b58970f",
      ],
    });
    const account = ref(null);

    /* Connect */

    const connectWallet = async () => {
      try {
        // const modal = new WalletConnectModal({ projectId });

        const provider = await UniversalProvider.init({
          projectId: projectId,
          logger: "info",
          relayUrl: "wss://relay.walletconnect.com",
          metadata: {
            name: "WalletConnect Universal Provider",
            description: "WalletConnect Universal Provider Demo",
            url: "https://walletconnect.com/",
            icons: ["https://avatars.githubusercontent.com/u/37784886"],
          },
        });

        provider.on("display_uri", async (uri) => {
          console.log("display_uri", uri);
          web3Modal.closeModal();
          await web3Modal.openModal({ uri });
        });

        await provider.connect({
          namespaces: {
            eip155: {
              methods: [
                "eth_sendTransaction",
                "eth_signTransaction",
                "eth_sign",
                "personal_sign",
                "eth_signTypedData",
                "wallet_watchAsset",
                "wallet_addEthereumChain",
                "wallet_switchEthereumChain",
              ],
              chains: ["eip155:1111"],
              events: ["chainChanged", "accountsChanged"],
              rpcMap: { ["1111"]: "https://api.wemix.com" },
            },
          },
        });

        web3Modal.closeModal();

        const ethersWeb3Provider = new ethers.BrowserProvider(provider);
        const signer = await ethersWeb3Provider.getSigner();
        const account = signer.address;
        console.log("account", account);

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
