<!-- MessageList.svelte -->
<script>
  import "iconify-icon";
  let messages = [
    { id: 1, sender: "Alice", text: "Hi there!", timestamp: new Date() },
    { id: 2, sender: "Bob", text: "How are you?", timestamp: new Date() },
    {
      id: 3,
      sender: "Alice",
      text: "I'm good, thanks!",
      timestamp: new Date(),
    },
    { id: 4, sender: "Charlie", text: "See you later!", timestamp: new Date() },
  ];
</script>

<div class="flex flex-col gap-2 p-2">
  {#each messages as message}
    <div class="p-2 rounded-md bg-gray-100">
      <p class="font-bold">
        {message.sender} - {message.timestamp.toLocaleString()}
      </p>
      <p>{message.text}</p>
    </div>
  {/each}
</div>
