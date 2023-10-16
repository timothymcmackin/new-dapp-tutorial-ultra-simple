<script>
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import { NetworkType } from "@airgap/beacon-sdk";
  import { TezosToolkit } from "@taquito/taquito";

  const contractAddress = "KT1AQw4eA5EfBCG5qvKFHia5jy8A4y4qQeEN";
  const rpcUrl = "https://ghostnet.ecadinfra.com";
  const Tezos = new TezosToolkit(rpcUrl);
  const network = NetworkType.GHOSTNET;

  let wallet;
  let address;
  let balance;

  let depositAmount = 1;
  let depositButtonActive = false;
  let depositButtonLabel = "Deposit";

  let withdrawButtonActive = true;
  let withdrawButtonLabel = "Withdraw";

  const getBalance = async (walletAddress) => {
    const balanceMutez = await Tezos.tz.getBalance(walletAddress);
    balance = balanceMutez.div(1000000).toFormat(2);
  }

  const connectWallet = async () => {
    const newWallet = new BeaconWallet({
      name: "Simple dApp tutorial",
      preferredNetwork: network,
    });
    await newWallet.requestPermissions({
      network: { type: network, rpcUrl },
    });
    address = await newWallet.getPKH();
    await getBalance(address);
    wallet = newWallet;
    depositButtonActive = true;
  };

  const disconnectWallet = () => {
    wallet.client.clearActiveAccount();
    wallet = undefined;
  };

  const deposit = async () => {
    depositButtonActive = false;
    depositButtonLabel = "Depositing...";
    Tezos.setWalletProvider(wallet);
    const contract = await Tezos.wallet.at(contractAddress);
    await contract.methods.deposit('').send({
      gasLimit: 100000,
      fee: 100000,
      storageLimit: 1,
      amount: depositAmount,
    })
      .then((op) => {
        console.log(`Waiting for ${op.opHash} to be confirmed...`);
        return op.confirmation(3).then(() => op.opHash);
      })
      .then((hash) => console.log(`Operation injected: https://ghost.tzstats.com/${hash}`))
      .catch((error) => console.log(`Error: ${JSON.stringify(error, null, 2)}`));

    await getBalance(address);
    depositButtonActive = true;
    depositButtonLabel = "Deposit";
  }

  const withdraw = async () => {
    withdrawButtonActive = false;
    withdrawButtonLabel = "Withdrawing...";

    Tezos.setWalletProvider(wallet);
    const contract = await Tezos.wallet.at(contractAddress);
    await contract.methods.withdraw('').send({
      gasLimit: 100000,
      fee: 100000,
      storageLimit: 1,
      amount: 0,
    })
      .then((op) => {
        console.log(`Waiting for ${op.opHash} to be confirmed...`);
        return op.confirmation(3).then(() => op.opHash);
      })
      .then((hash) => console.log(`Operation injected: https://ghost.tzstats.com/${hash}`))
      .catch((error) => console.log(`Error: ${JSON.stringify(error, null, 2)}`));

    await getBalance(address);
    withdrawButtonActive = true;
    withdrawButtonLabel = "Withdraw";
  }


</script>

<main>
  <div>
  </div>
  <h1>Tezos bank dApp</h1>

  <div class="card">
    {#if wallet}
      <p>The address of the connected wallet is {address}.</p>
      <p>Its balance in tez is {balance}.</p>
      <p>
        Deposit tez:
        <input type="number" bind:value={depositAmount} min="1" max="100" />
        <input type="range" bind:value={depositAmount} min="1" max="100" />
        <button on:click={deposit} disabled={!depositButtonActive}> {depositButtonLabel} </button>
      </p>
      <p>
        Withdraw tez:
        <button on:click={withdraw} disabled={!withdrawButtonActive}> {withdrawButtonLabel} </button>
      </p>
      <p>
        <button on:click={disconnectWallet}> Disconnect wallet </button>
      </p>
    {:else}
      <button on:click={connectWallet}> Connect wallet </button>
    {/if}
  </div>
</main>

<style>
</style>
