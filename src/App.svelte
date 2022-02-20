<script lang="ts">
  import { onMount } from "svelte";
  import { Connection } from "@solana/web3.js";
  import { Provider, web3 } from "@project-serum/anchor";
  import { fade } from "svelte/transition";
  import Button from "./components/Button.svelte";
  import Header from "./components/CardHeader.svelte";

  import { candyMachineState, userState } from "./lib/store";
  import {
    getCandyMachineState,
    checkWalletConnected,
    getUserBalance,
    existsOwnerSPLToken,
  } from "./lib/state-helpers";
import { time_ranges_to_array } from "svelte/internal";

  /***********************************/
  // Customise the app by changing the following variables.
  const TITLE = "Sol Visitors Are Landing – OFFICIAL MINT SITE";
  const DESCRTIPTION = "Official mint site for Sol Visitors NFT project on Solana blockchain. This Imprint NFT entitles you to minimum 1 Visitor airdroppped from main collection and also grants whitelist access.";
  const HEADER_TITLE = "Sol Visitors";
  const HEADER_LINK = "https://solvisitors.com";
  // Your image or GIF needs to be in the /public folder for this to work
  const IMAGE_LINK = "/images/SolVisitors_Imprint_NFT.gif";
  /***********************************/

  let { solana } = window as any;
  const rpcUrl = import.meta.env.VITE_APP_SOLANA_RPC_HOST?.toString();
  const cluster = import.meta.env.VITE_APP_SOLANA_NETWORK?.toString();
  const candyMachineId = import.meta.env.VITE_APP_CANDY_MACHINE_ID?.toString();
  const opts = { preflightCommitment: "processed" };

  let siteLoading = true;
  let errorOcurred = false;
  let connection: Connection;
  let provider: Provider;
  let candyMachinePublicKey: web3.PublicKey;

  $: itemsRedeemed = $candyMachineState?.state.itemsRedeemed;
  $: itemsAvailable = $candyMachineState?.state.itemsAvailable;

  function checkEnvironmentVariables() {
    // Check if populated
    if (!rpcUrl || !candyMachineId || !cluster) {
      if (!rpcUrl) {
        console.error("RPC URL not populated");
      }
      if (!candyMachineId) {
        console.error("Candy Machine ID not populated");
      }
      if (!cluster) {
        console.error("Environment not populated");
      }
      return true;
    }
    if (candyMachineId.length < 32 || candyMachineId.length > 44) {
      console.error(
        "Candy Machine Public Key is invalid. Enter a length in-between 32 and 44 characters"
      );
      return true;
    }
    return false;
  }

  onMount(async () => {
    solana = (window as any).solana;
    // Check if environement variables are populated
    errorOcurred = checkEnvironmentVariables();
    if (errorOcurred) {
      return;
    }

    // If env variables populated, create provider, PK and connection
    connection = new Connection(rpcUrl);
    provider = new Provider(
      connection,
      solana,
      opts.preflightCommitment as web3.ConfirmOptions
    );
    candyMachinePublicKey = new web3.PublicKey(candyMachineId);

    // Get candy machine state
    $candyMachineState = await getCandyMachineState(
      candyMachinePublicKey,
      provider
    );

    // Establish connection to wallet
    if (solana?.isPhantom) {
      $userState.walletPublicKey = await checkWalletConnected(solana);
      if ($userState.walletPublicKey) {
        // Get User Balance
        $userState.userBalance = await getUserBalance(
          $userState.walletPublicKey,
          connection
        );
        // If whitelist config populated, check if user is whitelisted (ie. check if they have token)
        if ($candyMachineState.state.whitelistMintSettings) {
          $userState.isWhiteListed = await existsOwnerSPLToken(
            $userState.walletPublicKey,
            connection,
            $candyMachineState.state.whitelistMintSettings?.mint
          );
        }
      }
    }

    // Stop loading
    siteLoading = false;
  });
</script>

<main class="h-screen">
  <!-- Error section -->
  {#if errorOcurred}
    <div class=" h-full flex">
      <div class="m-auto">
        An error occurred. Please check if your environment variables have been
        populated correctly and redeploy the applcation.
      </div>
    </div>
    <!-- Loading Section -->
  {:else if siteLoading && !errorOcurred}
    <div class=" h-full flex">
      <div class="lds-hourglass m-auto" />
    </div>
  {:else}
    <!-- Menu Bar -->
    {#if HEADER_TITLE}
    <div class="text-white/40  my-2">
      <a href="https://solvisitors.com/" target="_blank" class="text-indigo-200  underline underline-offset-4 decoration-2 font-mono">SolVisitors.com</a> • <a href="https://twitter.com/SolVisitors" target="_blank" class="text-sky-400 underline underline-offset-4 decoration-2 font-mono">Twitter</a> • <a href="https://discord.gg/TF7zW5q9Ur" target="_blank" class="text-indigo-500 underline underline-offset-4 decoration-2 font-mono">Discord</a> • <a href="https://solvisitors.com/get-started-with-nfts-and-defi-on-solana-blockchain/" target="_blank" class="text-fuchsia-500 underline underline-offset-4 decoration-2 font-mono">I'm New. Help?</a>
    </div>
    {/if}
    <!-- Card -->
    <div
      class=" max-w-lg mx-auto bg-black/40 rounded-lg my-6  border-2 border-indigo-500/50"
      transition:fade
    >
      <!-- Top Bar -->
      <Header />
      <hr class="border-indigo-500/50" />
      <!-- Main Body -->
      <div class="p-4">
        <h1
          class=" text-lg sm:text-2xl font-mono font-bold py-2 tracking-wider text-white"
        >
        Sol Visitors Are Landing!🛸<br />
        OFFICIAL MINT SITE<br />
        <!-- <small class="text-indigo-200/70">mint.solvisitors.com</small> -->
      </h1>
        <img src={IMAGE_LINK} alt="{TITLE}" class=" w-1/2 mx-auto mb-5 rounded-lg shadow-xl border-2 border-white/30" />
        <div class="text-md sm:text-md pb-5 text-white ">
          <!-- {DESCRTIPTION} -->
          <h3 class="m-4"><strong>Public (Not Whitelist) Sale Begins</strong><br />
            <small>{new Date($candyMachineState?.state.goLiveDate?.toNumber() * 1000)}</small></h3>

            <h2 class="m-4 text-lg">First Wave: The Imprints</h2>

          You are about to mint a <abbr title="Non-Fungible Token">NFT</abbr> on the Solana blockchain. We hope you enjoy the artwork and will do your best to be a good steward of the Visitor community. <em>Welcome Aboard!</em><br /><br />

          {#if $candyMachineState.state.whitelistMintSettings && 
            new Date($candyMachineState?.state.goLiveDate?.toNumber() * 1000) > new Date}
            {#if $userState.isWhiteListed}
            <div class="p-2 shadow-lg bg-indigo-900	rounded-lg">
              <span class="my-auto text-gray-200 text-sm">
                <strong>Congratulations, you have the whitelist token!</strong><br />
                <img src="images/Sol_Visitors_BEACON_coin.png" alt="Whitelist Token" width="18" height="18" style="width: 18px; height: 18px; margin: -2px 4px 0 0; display: inline;" class="" /> <span class="text-xs font-mono tracking-wider">{$candyMachineState.state.whitelistMintSettings?.mint}</span><br />
                <em>One whitelist token is required per mint.</em></span>
              </div>
            {/if}
            {#if !$userState.isWhiteListed}
            <div class="p-2 shadow-lg bg-orange-900/40	rounded-lg">
              <span class="my-auto text-gray-200 text-sm">
                <strong>You need a whitelist token to mint at this time.</strong><br />
                <img src="images/Sol_Visitors_BEACON_coin.png" alt="Whitelist Token" width="18" height="18" style="width: 18px; height: 18px; margin: -2px 4px 0 0; display: inline;" class="" /> <span class="text-xs font-mono tracking-wider">{$candyMachineState.state.whitelistMintSettings?.mint}</span><br />
                <em>You can try asking for one in the <a href="https://discord.gg/TF7zW5q9Ur" target="_blank" class="underline underline-offset-2">Discord</a>.</em></span>
              </div>
            {/if}
          {/if}
          <div class="">
          <br /><strong>Price:</strong> 0.42 SOL + network fees
        </div>
        </div>

        <Button {connection} />

        <div class="pt-3 text-md sm:text-md pb-5 text-white ">
        <strong>Quantity:</strong> {itemsAvailable - itemsRedeemed} of {itemsAvailable} remaining.
        </div>



          {#if $userState.solanaExplorerLink}
          <div class="flex flex-col pt-3">
            <a
              href={$userState.solanaExplorerLink}
              target="_blank"
              class="text-purple-700 font-semibold  p-1"
              >View on Solana Explorer</a
            >
          </div>
          {/if}
        

        <a href="https://twitter.com/civickey/status/1478456202069565443" target="_blank" rel="noopener"><img class="m-2 w-40 mx-auto" src="images/Civic-Verified-Logo-768x208.png" alt="Verified by Civic"></a>  

      </div>
    </div>
      <a href="https://github.com/alvinsga/candy-machine-v2" target="_blank" rel="noopener" class="text-white/50 m-8 text-sm">❤️ Thank you to alvinsga for Candy Machine V2 Frontend</a>
      <br /><br />
  {/if}
</main>
