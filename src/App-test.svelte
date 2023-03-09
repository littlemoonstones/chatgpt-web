<!-- App.svelte -->
<script>
  import { Router, Route } from "svelte-routing";
  import MenuButton from "./lib/MenuButton.svelte";
  import ConversationList from "./lib/ConversationList.svelte";
  import PageLinks from "./lib/PageLinks.svelte";
  import MessageList from "./lib/MessageList.svelte";
  import SendMessageForm from "./lib/SendMessageForm.svelte";
  import Home from "./lib/HomePage.svelte";
  import About from "./lib/AboutPage.svelte";

  let showDrawer = false;

  function toggleDrawer() {
    showDrawer = !showDrawer;
  }
</script>

<div class="flex flex-col h-screen">
  <!-- Mobile app bar -->
  <div
    class="flex items-center justify-between px-4 py-2 bg-blue-500 text-white md:hidden"
  >
    <MenuButton on:click={toggleDrawer} />
    <h1 class="text-lg font-bold">My Chat App</h1>
    <button class="p-1 rounded-full bg-white text-blue-500">
      <svg class="w-6 h-6" viewBox="0 0 24 24">
        <path
          fill="currentColor"
          d="M19,11H13V5c0-1.1-0.9-2-2-2s-2,0.9-2,2v6H5c-1.1,0-2,0.9-2,2s0.9,2,2,2h4v6c0,1.1,0.9,2,2,2s2-0.9,2-2v-6h6c1.1,0,2-0.9,2-2S20.1,11,19,11z"
        />
      </svg>
    </button>
  </div>

  <!-- Main content -->
  <div class="flex flex-grow">
    <!-- Drawer -->
    <div
      class="md:w-1/5 bg-gray-100 border-gray-200 overflow-hidden transition-all duration-300 transform md:translate-x-0"
      class:translate-x-[-100%]={!showDrawer}
      class:z-10={showDrawer}
      class:w-72={showDrawer}
      class:w-0={!showDrawer}
    >
      <div class="">
        <div class="h-96 overflow-y-auto">
          <ConversationList />
        </div>
        <div class="divider" />
        <div class="">
          <PageLinks/>
        </div>
      </div>
    </div>

    <!-- Messages -->
    <div class="w-full md:w-4/5 flex flex-col">
      <div class="flex-grow overflow-y-auto">
        <MessageList />
      </div>
      <SendMessageForm />
    </div>
  </div>

  <!-- Routing -->
  <!-- <Router>
    <Route path="/" component={Home} />
    <Route path="/about" component={About} />
  </Router> -->
</div>
