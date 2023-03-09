<script lang="ts">
  import Icon from "@iconify/svelte";
  import SendMessageForm from "./lib/SendMessageForm.svelte";
  import Conversation from "./lib/Conversation.svelte";
  import SidebarNew from "./lib/SidebarNew.svelte";
  import { addChat } from "./lib/Storage.svelte";
  import { apiKeyStorage, chatsStorage } from "./lib/Storage.svelte";

  // https://icon-sets.iconify.design/?query=plus
  // let conversations = [
  //   { id: 1, name: "Alice", lastMessage: "Hi there!" },
  //   { id: 2, name: "Bob", lastMessage: "How are you?" },
  //   { id: 3, name: "Charlie", lastMessage: "See you later!" },
  //   { id: 4, name: "Charlie", lastMessage: "See you later!" },
  //   { id: 5, name: "Charlie", lastMessage: "See you later!" },
  //   { id: 6, name: "Charlie", lastMessage: "See you later!" },
  //   { id: 7, name: "Charlie", lastMessage: "See you later!" },
  //   { id: 8, name: "Charlie", lastMessage: "See you later!" },
  // ];
  let activeChatId = 3;
  $: sortedChats = $chatsStorage.sort((a, b) => b.id - a.id);
  $: apiKey = $apiKeyStorage;
</script>

<div class="drawer drawer-mobile">
  <input id="my-drawer-2" type="checkbox" class="drawer-toggle" />
  <div class="drawer-content flex flex-col">
    <!-- Navbar START -->

    <div
      class="
    sticky top-0 z-30 flex h-16 w-full justify-center bg-opacity-90 backdrop-blur transition-all duration-100 
    bg-base-100 text-base-content shadow-sm lg:hidden
    "
    >
      <nav class="navbar w-full">
        <div class="flex flex-1 justify-between md:gap-1 lg:gap-2">
          <span>
            <label
              for="my-drawer-2"
              class="btn btn-primary drawer-button lg:hidden"
              ><Icon icon="material-symbols:menu-rounded" /></label
            >
          </span>
          <div class="flex items-center gap-2 lg:hidden">My chat</div>
          <div
            class="mr-2"
            on:click|preventDefault={() => {
              activeChatId = addChat();
            }}
          >
            <Icon icon="material-symbols:add" width="26" height="26" />
          </div>
        </div>
      </nav>
    </div>
    <!-- Navbar END -->

    <!-- <div class=" w-10 bg-red-100">hello world</div> -->
    <!-- Page content START -->
    <!-- <div class="flex flex-1 flex-col justify-between bg-slate-700"> -->
    <div class="h-screen overflow-hidden">
      <div class="h-full flex flex-col justify-between bg-slate-700">
        <Conversation bind:chatId={activeChatId} />
        <!-- <div class="flex-1 overflow-y-auto">
          <div class="p-20 bg-green-500" />
          <div class="p-20 bg-orange-500" />
          <div class="p-20 bg-green-500" />
          <div class="p-20 bg-orange-500" />
          <div class="p-20 bg-green-500" />
          <div class="p-20 bg-orange-500" />
          <div class="p-20 bg-green-500" />
          <div class="p-20 bg-orange-500">
            last 1
          </div>
        </div>
        <div class="w-full h-48 bg-red-500" /> -->
      </div>
      <!-- Page content END -->
    </div>
  </div>

  <div class="drawer-side">
    <SidebarNew bind:apiKey bind:sortedChats bind:activeChatId />
  </div>

  <!-- Put this part before </body> tag -->
</div>
