<script lang="ts">
  import Icon from "@iconify/svelte";

  import { addChat, clearChats } from "./Storage.svelte";
  import { exportAsMarkdown } from "./Export.svelte";
  import type { Chat } from "./Types.svelte";

  export let activeChatId: number;
  export let sortedChats: Chat[];
  export let apiKey: string;
</script>

<label for="my-drawer-2" class="drawer-overlay" />
<!-- <ul class="menu w-72 overflow-auto bg-base-100 flex flex-col p-0 px-4"> -->
<ul class="menu w-72 bg-base-100 p-0 px-4">
  <!-- Sidebar content here -->
  {#if sortedChats.length === 0}
    <li><a class="panel-block" href={"#"}>No chats yet...</a></li>
  {:else}
    <li class="mt-4 border rounded-lg">
      <a href={"#"} on:click|preventDefault={() => {
        activeChatId = addChat();
      }}><Icon icon="material-symbols:add" />New Chat</a>
    </li>
    <li />
    <li class="menu-title">
      <span class="flex flex-row"><Icon icon="clarity:talk-bubbles-line" /> Chats</span>
    </li>
    <div class="h-1/3 overflow-auto block">
      {#each sortedChats as chat}
        <li>
          <a
            href={"#"}
            class:is-disabled={!apiKey}
            class:has-background-light={activeChatId === chat.id}
            on:click|preventDefault={() => (activeChatId = chat.id)}
            > {chat.name || `Chat ${chat.id}`}</a
          >
        </li>
      {/each}
    </div>
  {/if}
  <li />
  <li>
    <a
      href={"#"}
      on:click|preventDefault={() => {
        clearChats();
        activeChatId = null;
      }}><Icon icon="mdi:delete-sweep" />Clean All</a
    >
  </li>
  <li>
    <a href={"#"} on:click|preventDefault={exportAsMarkdown(activeChatId)}
      ><Icon icon="ic:outline-ios-share" />Export</a
    >
  </li>
  <!-- <li>
    <a> <Icon icon="ic:round-key" />API Key</a>
  </li> -->
  <li>
    <label for="my-modal-6"
      ><a class="flex flex-row" href={"#"}
        ><Icon icon="ic:round-key" class="text-xl mr-2" />API Key</a
      ></label
    >
  </li>
</ul>
<!-- <aside class="menu">
  <p class="">Chats</p>
  <ul class="menu bg-base-100 w-56">
    {#if sortedChats.length === 0}
      <li><a class="panel-block" href={"#"}>No chats yet...</a></li>
    {:else}
      <li>
        <ul>
          {#each sortedChats as chat}
            <li>
              <a
                href={"#"}
                class="panel-block"
                class:is-disabled={!apiKey}
                class:has-background-light={activeChatId === chat.id}
                on:click|preventDefault={() => (activeChatId = chat.id)}>{chat.name || `Chat ${chat.id}`}</a
              >
            </li>
          {/each}
        </ul>
      </li>
    {/if}
  </ul>
  <p class="">Actions</p>
  <ul class="">
    <li>
      <a
        href={"#"}
        class="panel-block"
        class:is-disabled={!apiKey}
        class:has-background-light={!activeChatId}
        on:click|preventDefault={() => {
          activeChatId = null;
        }}><span class="greyscale mr-2">🔑</span> API key</a
      >
    </li>
    <li>
      <a
        href={"#"}
        class="panel-block"
        class:is-disabled={!apiKey}
        on:click|preventDefault={() => {
          activeChatId = addChat();
        }}><span class="greyscale mr-2">➕</span> New chat</a
      >
    </li>
    <li>
      <a
        href={"#"}
        class="panel-block"
        class:is-disabled={!apiKey}
        on:click|preventDefault={() => {
          clearChats();
          activeChatId = null;
        }}><span class="greyscale mr-2">🗑️</span> Clear chats</a
      >
    </li>
    {#if activeChatId }
    <li>
      <a
        href={"#"}
        class="panel-block"
        class:is-disabled={!apiKey}
        on:click|preventDefault={() => {
          exportAsMarkdown(activeChatId);
        }}><span class="greyscale mr-2">📥</span> Export chat</a
      >
    </li>
    {/if}
  </ul>
</aside> -->

<input type="checkbox" id="my-modal-6" class="hidden modal-toggle" />
<div class="modal modal-bottom sm:modal-middle">
  <div class="modal-box">
    <h3 class="font-bold text-lg">Enter Your API Key</h3>
    <div class="form-control w-full max-w-xs">
      <input
        type="text"
        placeholder="Type here"
        class="input input-bordered w-full max-w-xs"
      />
    </div>
    <div class="flex flex-row justify-between">
      <div class="modal-action">
        <label for="my-modal-6" class="btn">Cancel</label>
      </div>
      <div class="modal-action">
        <label for="my-modal-6" class="btn btn-primary">Save</label>
      </div>
    </div>
  </div>
</div>
