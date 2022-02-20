<script>
  import { userState } from "../lib/store";
  import { LAMPORTS_PER_SOL } from "@solana/web3.js";
  let currentDate = new Date();
</script>

<div class="justify-between flex p-3">
  <div class="text-white">
    <!--{currentDate.toString()}-->
  </div>
  <!-- {#if $userState.isWhiteListed}
    <div class="flex mr-auto">
      <span class="my-auto text-gray-200 text-sm">
        <img src="images/Sol_Visitors_BEACON_coin.png" alt="Whitelist Token" width="24" height="24" style="width: 24px; height: 24px; display: inline-block; margin: -2px 4px 0 0;" class="" />
        You have a Whitelist Token!</span>
    </div>
  {/if} -->
  <div class="flex">
    {#if !$userState.walletPublicKey}
      <span class="my-auto mr-2 rounded-full h-2 w-2 bg-gray-500" />
      <span class=" my-auto text-gray-600 text-sm"> Not Connected </span>
    {:else}
      <div class=" flex flex-col">
        <div class="flex">
          <span class="my-auto mr-2 rounded-full h-2 w-2 bg-green-500" />
          <span class=" my-auto text-gray-200 text-sm">
            {$userState.walletPublicKey?.slice(0, 4)}...{$userState.walletPublicKey?.slice($userState.walletPublicKey.length-4)}
          </span>
        </div>
        <div class=" text-xs text-gray-400 text-right ">
          {($userState?.userBalance / LAMPORTS_PER_SOL).toFixed(2)} SOL
        </div>
      </div>
    {/if}
  </div>
</div>
