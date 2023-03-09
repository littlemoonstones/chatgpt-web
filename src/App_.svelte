<script lang="ts">
  import Navbar from "./lib/Navbar.svelte";
  import Sidebar from "./lib/Sidebar.svelte";
  import Home from "./lib/Home.svelte";
  import Chat from "./lib/Chat.svelte";
  import ChatNew from "./lib/ChatNew.svelte";
  import Footer from "./lib/Footer.svelte";
  import SidebarNew from "./lib/SidebarNew.svelte";

  import { apiKeyStorage, chatsStorage } from "./lib/Storage.svelte";

  $: sortedChats = $chatsStorage.sort((a, b) => b.id - a.id);
  $: apiKey = $apiKeyStorage;

  // Check if the API key is passed in as a "key" query parameter - if so, save it
  const urlParams = new URLSearchParams(window.location.search);
  if (urlParams.has("key")) {
    apiKeyStorage.set(urlParams.get("key")!);
  }

  let activeChatId: number;

  // const menuButton = document.getElementById('menuButton');
  // const firstColumn = document.getElementById('firstColumn');
  let firstColumnVisible = false;
  function toggleFirstColumn() {
    firstColumnVisible = !firstColumnVisible;
  }
  // const showUP = ()=>{
  //   firstColumn.classList.toggle('hidden');
  // }
  // menuButton.addEventListener('click', () => {
    
  // });
</script>

<!-- <Navbar /> -->
<div class="flex flex-col h-screen">
  <!-- app bar for mobile view -->
  <div class="bg-gray-800 text-white p-4 flex justify-between items-center md:hidden">
    <button id="menuButton" class="focus:outline-none" on:click={toggleFirstColumn}>
      <svg class="h-6 w-6 fill-current" viewBox="0 0 24 24">
        <path d="M4 6h16v2H4zm0 5h16v2H4zm0 5h16v2H4z"></path>
      </svg>
    </button>
    <span class="text-xl font-semibold">Chat Rooms</span>
    <button class="focus:outline-none">
      <svg class="h-6 w-6 fill-current" viewBox="0 0 24 24">
        <path d="M7 10l5 5 5-5z"></path>
      </svg>
    </button>
  </div>
  
  <!-- main content -->
  <div class="flex flex-row flex-1">
    <!-- first column -->
    {#if firstColumnVisible}
      <div class="w-1/6 bg-gray-200 md:block">
        <!-- list of chat rooms -->
      </div>
    {/if}

    <!-- second column -->
    <div class="w-full bg-white">
      <div class="flex flex-col justify-end h-full">
        <div class="flex-1 p-4 overflow-y-auto">
          <!-- chat messages go here -->
        </div>
        <div class="flex p-4">
          <input type="text" class="flex-1 px-4 py-2 rounded-lg border-gray-300 focus:outline-none focus:border-blue-500" placeholder="Type a message...">
          <button class="ml-2 px-4 py-2 font-semibold text-white bg-blue-500 rounded-lg">Send</button>
        </div>
      </div>
    </div>
  </div>
</div>

<!-- <div class="flex">
  <div class="w-1/6 h-screen bg-gray-200"></div>
  <div class="w-5/6 h-screen bg-white">
    <div class="flex flex-col justify-end h-full">
      <div class="flex-1 p-4 overflow-y-auto">
        chat messages go here
      </div>
      <div class="flex p-4">
        <input type="text" class="flex-1 px-4 py-2 rounded-lg border-gray-300 focus:outline-none focus:border-blue-500" placeholder="Type a message...">
        <button class="ml-2 px-4 py-2 font-semibold text-white bg-blue-500 rounded-lg">Send</button>
      </div>
    </div>
  </div>
</div> -->
<!-- <section class="section container">
  <div class="container is-fullhd">
  <div class="grid grid-cols-5 is-fullhd">
      <div class="column col-span-1 is-one-fifth">
        <Sidebar bind:apiKey bind:sortedChats bind:activeChatId />
        <SidebarNew bind:apiKey bind:sortedChats bind:activeChatId />
      </div>
      <div class="column col-span-4 h-full is-four-fifths">
        {#if activeChatId}
          <ChatNew bind:chatId={activeChatId} />
        {:else}
          <Home bind:activeChatId />
        {/if}
      </div>
  </div>
</section> -->

<!-- <Footer /> -->
