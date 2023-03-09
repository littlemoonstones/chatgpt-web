<script>
  export let conversations = [];
</script>

<div class="flex flex-col h-screen">
  <div class="flex flex-row h-full">
    <div class="w-1/6 border-r-2">
      <div class="h-4/6 overflow-y-auto">
        <!-- list of conversations -->
        {#each conversations as conversation}
          <div>{conversation}</div>
        {/each}
      </div>
      <div class="h-2/6 flex justify-center items-center">
        <a href="/">Home</a>
        <a href="/about">About</a>
      </div>
    </div>
    <div class="w-5/6">
      <!-- messages -->
      <div class="h-5/6 overflow-y-auto" />
      <!-- input and send button -->
      <div class="h-1/6 flex flex-row">
        <input type="text" class="w-full p-2" placeholder="Type a message..." />
        <button
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
          >Send</button
        >
      </div>
    </div>
  </div>
</div>
