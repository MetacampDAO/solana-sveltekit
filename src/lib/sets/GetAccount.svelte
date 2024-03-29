<script lang="ts">
  import { walletStore } from "@svelte-on-solana/wallet-adapter-core";
  import {
    PublicKey,
    type ParsedAccountData,
    type AccountInfo,
    type RpcResponseAndContext,
    type Connection,
  } from "@solana/web3.js";

  import {
    getAccountInfo,
    getParsedAccountInfo,
    subscribeAccountInfo,
  } from "../utils/systemProgram";

  import { cluster, connectedCluster, notify } from "$lib/stores";
  import Button from "../components/Button.svelte";

  // Input/Output variables used
  let pubKey: PublicKey | string;
  $: pubKey;
  let accountData:
    | RpcResponseAndContext<AccountInfo<Buffer | ParsedAccountData> | null>
    | AccountInfo<Buffer>
    | null;
  $: accountData;
  let subscribedAccountInfoChange: number;

  // Get AccountInfo function
  const getAccountInfoHandler = async (
    connection: Connection,
    pubKey: PublicKey
  ) => {
    notify("Getting Account Info");

    // Get accountInfo
    accountData = await getAccountInfo(connection, pubKey);
  };

  // Get ParsedAccountData function
  const getParsedAccountDataHandler = async (
    connection: Connection,
    pubKey: PublicKey
  ) => {
    notify("Getting Parsed Account Info");

    // Get Parsed AccountInfo
    accountData = await getParsedAccountInfo(connection, pubKey);
  };

  // Subscribe AccountData function
  const subscribeAccountInfoDataHandler = async (
    connection: Connection,
    pubKey: PublicKey
  ) => {
    // Subscribe accountInfo
    subscribedAccountInfoChange = subscribeAccountInfo(connection, pubKey);
  };
</script>

<!-- HTML + SVELTE -->
<section
  class="w-2/3 p-4 space-y-4 text-black rounded-md bg-dark dark:text-white"
>
  <h1>Get/Subscribe to Account</h1>
  <div>
    <!-- User Input -->
    <div>
      <div>
        <label for="Account" />
        <input
          class="w-full mb-2 text-black"
          bind:value={pubKey}
          placeholder="Enter address ..."
        />
      </div>

      <div class="flex flex-wrap items-center">
        <div class="mt-1 mb-1 mr-2">
          <Button
            label="Get AccountInfo"
            onClick={() =>
              getAccountInfoHandler($connectedCluster, new PublicKey(pubKey))}
          />
        </div>

        <div class="mt-1 mb-1 mr-2">
          <Button
            label="Get ParsedAccountInfo"
            onClick={() =>
              getParsedAccountDataHandler(
                $connectedCluster,
                new PublicKey(pubKey)
              )}
          />
        </div>

        <div class="mt-1 mb-1 mr-2">
          <Button
            label="Subscribe Account"
            onClick={() =>
              subscribeAccountInfoDataHandler(
                $connectedCluster,
                new PublicKey(pubKey)
              )}
          />
        </div>

        <div class="mt-1 mb-1 mr-2">
          <a
            href="https://solana.fm/address/{pubKey}?cluster={$cluster}"
            target="_blank"
            rel="noopener noreferrer"
          >
            <Button label="Check Explorer" />
          </a>
        </div>

        <div class="mt-1 mb-1 mr-2">
          <Button
            label="Copy Wallet"
            styling="bg-blue-600"
            onClick={() => (pubKey = $walletStore.publicKey || "")}
          />
        </div>
      </div>
    </div>

    <!-- Response Output -->
    <div>
      {#if subscribedAccountInfoChange !== undefined}
        <div>Subscription ID: {subscribedAccountInfoChange}</div>
      {/if}
    </div>
    <div>
      {#await accountData}
        <p>waiting</p>
      {:then account}
        {#if account !== undefined}
          <pre class="break-words">{JSON.stringify(account, null, 2)}</pre>
        {/if}
      {:catch error}
        <p>No data</p>
      {/await}
    </div>
  </div>
</section>
