<!-- ConversationList.svelte -->
<script>
  let conversations = [
    { id: 1, name: "Alice", lastMessage: "Hi there!" },
    { id: 2, name: "Bob", lastMessage: "How are you?" },
    { id: 3, name: "Charlie", lastMessage: "See you later!" },
    { id: 4, name: "Charlie", lastMessage: "See you later!" },
    { id: 5, name: "Charlie", lastMessage: "See you later!" },
    { id: 6, name: "Charlie", lastMessage: "See you later!" },
    { id: 7, name: "Charlie", lastMessage: "See you later!" },
    { id: 8, name: "Charlie", lastMessage: "See you later!" },
  ];
</script>

<div class="">
  {#each conversations as conversation}
    <div class="p-2 hover:bg-gray-200">
      <h2 class="font-bold">{conversation.name}</h2>
      <p>{conversation.lastMessage}</p>
    </div>
  {/each}
</div>
