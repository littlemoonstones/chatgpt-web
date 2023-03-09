<script lang="ts">
  //import { fetchEventSource } from "@microsoft/fetch-event-source";
  import SendMessageForm from "./SendMessageForm.svelte";
  import Icon from "@iconify/svelte";
  import {
    apiKeyStorage,
    chatsStorage,
    addMessage,
    clearMessages,
  } from "./Storage.svelte";
  import type { Request, Response, Message, Settings } from "./Types.svelte";

  import { afterUpdate, onMount } from "svelte";
  import SvelteMarkdown from "svelte-markdown";

  export let chatId: number;
  let updating: boolean = false;

  let input: HTMLInputElement;
  let settings: HTMLDivElement;

  const settingsMap: Settings[] = [
    {
      key: "temperature",
      name: "Sampling Temperature",
      default: 1,
      type: "number",
    },
    {
      key: "top_p",
      name: "Nucleus Sampling",
      default: 1,
      type: "number",
    },
    {
      key: "n",
      name: "Number of Messages",
      default: 1,
      type: "number",
    },
    {
      key: "max_tokens",
      name: "Max Tokens",
      default: 0,
      type: "number",
    },
    {
      key: "presence_penalty",
      name: "Presence Penalty",
      default: 0,
      type: "number",
    },
    {
      key: "frequency_penalty",
      name: "Frequency Penalty",
      default: 0,
      type: "number",
    },
  ];

  $: chat = $chatsStorage.find((chat) => chat.id === chatId);

  const token_price = 0.000002; // $0.002 per 1000 tokens

  // Scroll to the bottom of the chat on update
  afterUpdate(() => {
    // Scroll to the bottom of the page after any updates to the messages array
    window.scrollTo(0, document.body.scrollHeight);
    // input.focus();
  });

  // Marked options
  const markedownOptions = {
    gfm: true,
    breaks: true,
  };

  const sendRequest = async (messages: Message[]): Promise<Response> => {
    // Send API request
    /*
    // Not working yet: a way to get the response as a stream
    await fetchEventSource("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        Authorization:
          `Bearer ${$apiKeyStorage}`,
        "Content-Type": "text/event-stream",
      },
      body: JSON.stringify(request),
      onmessage(ev) {
        console.log(ev);
      },
      onerror(err) {
        throw err;
      },
    });
    */
    // Show updating bar
    updating = true;

    let response: Response;
    try {
      const request: Request = {
        model: "gpt-3.5-turbo",
        // Submit only the role and content of the messages, provide the previous messages as well for context
        messages: messages
          .map((message): Message => {
            const { role, content } = message;
            return { role, content };
          })
          // Skip error messages
          .filter((message) => message.role !== "error"),

        // Provide the settings by mapping the settingsMap to key/value pairs
        ...settingsMap.reduce((acc, setting) => {
          const value = (
            settings.querySelector(
              `#settings-${setting.key}`
            ) as HTMLInputElement
          ).value;
          if (value) {
            acc[setting.key] =
              setting.type === "number" ? parseFloat(value) : value;
          }
          return acc;
        }, {}),
      };
      response = await (
        await fetch("https://api.openai.com/v1/chat/completions", {
          method: "POST",
          headers: {
            Authorization: `Bearer ${$apiKeyStorage}`,
            "Content-Type": "application/json",
          },
          body: JSON.stringify(request),
        })
      ).json();
    } catch (e) {
      response = { error: { message: e.message } } as Response;
    }

    // Hide updating bar
    updating = false;

    return response;
  };

  const submitForm = async (recorded: boolean = false): Promise<void> => {
    // Compose the input message
    const inputMessage: Message = { role: "user", content: input.value };
    addMessage(chatId, inputMessage);

    // Clear the input value
    input.value = "";
    input.blur();

    // Resize back to single line height
    input.style.height = "auto";

    const response = await sendRequest(chat.messages);

    if (response.error) {
      addMessage(chatId, {
        role: "error",
        content: `Error: ${response.error.message}`,
      });
    } else {
      response.choices.map((choice) => {
        choice.message.usage = response.usage;
        addMessage(chatId, choice.message);
        // Use TTS to read the response, if query was recorded
        if (recorded && "SpeechSynthesisUtterance" in window) {
          const utterance = new SpeechSynthesisUtterance(
            choice.message.content
          );
          speechSynthesis.speak(utterance);
        }
      });
    }
  };

  const suggestName = async (): Promise<void> => {
    const suggestMessage: Message = {
      role: "user",
      content: "Can you give me a 5 word summary of this conversation's topic?",
    };
    addMessage(chatId, suggestMessage);

    const response = await sendRequest(chat.messages);

    if (response.error) {
      addMessage(chatId, {
        role: "error",
        content: `Error: ${response.error.message}`,
      });
    } else {
      response.choices.map((choice) => {
        choice.message.usage = response.usage;
        addMessage(chatId, choice.message);
        chat.name = choice.message.content;
        chatsStorage.set($chatsStorage);
      });
    }
  };

  const deleteChat = () => {
    if (confirm("Are you sure you want to delete this chat?")) {
      chatsStorage.update((chats) =>
        chats.filter((chat) => chat.id !== chatId)
      );
      chatId = null;
    }
  };

  const showSettings = () => {
    settings.classList.add("is-active");
  };

  const closeSettings = () => {
    settings.classList.remove("is-active");
  };

  const clearSettings = () => {
    settingsMap.forEach((setting) => {
      const input = settings.querySelector(
        `#settings-${setting.key}`
      ) as HTMLInputElement;
      input.value = "";
    });
  };
  let messageText: string;
</script>

<!-- <nav class="level chat-header">
  <div class="level-left">
    <div class="level-item">
      <p class="subtitle is-5">
        {chat.name || `Chat ${chat.id}`}
        <a
          href={"#"}
          class="greyscale ml-2 is-hidden has-text-weight-bold editbutton"
          title="Rename chat"
          on:click|preventDefault={() => {
            let newChatName = prompt(
              "Enter a new name for this chat",
              chat.name
            );
            if (newChatName) {
              chat.name = newChatName;
              chatsStorage.set($chatsStorage);
            }
          }}
        >
          ✏️
        </a>
        <a
          href={"#"}
          class="greyscale ml-2 is-hidden has-text-weight-bold editbutton"
          title="Suggest a chat name"
          on:click|preventDefault={suggestName}
        >
          💡
        </a>
        <a
          href={"#"}
          class="greyscale ml-2 is-hidden editbutton"
          title="Delete this chat"
          on:click|preventDefault={deleteChat}
        >
          🗑️
        </a>
      </p>
    </div>
  </div>

  <div class="level-right">
    <p class="level-item">
      <button
        class="button is-warning"
        on:click={() => {
          clearMessages(chatId);
        }}><span class="greyscale mr-2">🗑️</span> Clear messages</button
      >
    </p>
  </div>
</nav> -->
<div class="flex flex-col flex-grow overflow-y-auto">
  {#if chat.messages.length === 0}
  <div class="flex h-screen justify-center items-center">
    <h1 class="font-semibold text-gray-600 text-4xl mr-2">ChatGPT </h1>
    <span class="text-xs rounded-lg bg-yellow-200 text-yellow-800 py-1 px-1.5 uppercase">pay-per-use</span>
    <!-- <div class="text-center">
    </div> -->
  </div>
  {:else}
    {#each chat.messages as message}
      {#if message.role === "user"}
        <div class="flex bg-slate-700 text-white">
          <div class="flex mx-auto w-full md:w-2/3 px-3 py-8 md:py-10">
            <div
              class="w-8 h-8 bg-blue-500 rounded-full mr-3 md:mr-7 flex items-center justify-center"
            >
              <Icon class="text-2xl text-green-100" icon="mdi:user" />
            </div>
            <div class="flex-1">
              <SvelteMarkdown
                source={message.content}
                options={markedownOptions}
                renderers={{
                  /*code: Code*/
                }}
              />
            </div>
            <div>
              <a
                href={"#"}
                on:click={() => {
                  input.value = message.content;
                  input.focus();
                }}
              >
                <Icon icon="material-symbols:edit" />
              </a>
            </div>
          </div>
        </div>
      {:else if message.role === "system" || message.role === "error"}
        <div class="flex bg-gray-100">
          <div class="flex mx-auto w-full md:w-2/3 px-3 py-8 md:py-10">
            <div
              class="w-8 h-8 bg-emerald-500 rounded-full mr-3 md:mr-7 flex items-center justify-center"
            >
              <Icon
                class="text-2xl text-white"
                icon="fluent:brain-circuit-20-filled"
              />
            </div>
            <div class="text-red-900">
              <SvelteMarkdown
                source={message.content}
                options={markedownOptions}
                renderers={{
                  /*code: Code*/
                }}
              />
            </div>
          </div>
        </div>
      {:else}
        <div class="flex bg-gray-100 ">
          <div class="flex mx-auto w-full md:w-2/3 px-3 py-8 md:py-10">
            <div
              class="w-8 h-8 bg-emerald-500 rounded-full mr-3 md:mr-7 flex items-center justify-center"
            >
              <Icon
                class="text-2xl text-white"
                icon="fluent:brain-circuit-20-filled"
              />
            </div>
            <div class="flex-1 text-sm md:text-base prose prose-xl ml-0 pl-0">
              <SvelteMarkdown
                source={message.content}
                options={markedownOptions}
                renderers={{
                  /*code: Code*/
                }}
              />

              <!-- {#if message.usage}
                      <div class="collapse collapse-arrow ">
                        <input type="checkbox" class="peer" /> 
                        <div class="collapse-title bg-transparent text-primary-content text-black peer-checked:bg-secondary">
                          See how many tokens consumes
                        </div>
                        <div class="collapse-content bg-transparent text-primary-content peer-checked:bg-secondary peer-checked:text-secondary-content"> 
                            <p>
                                This message was generated using <span
                                class="font-bold">{message.usage.total_tokens}</span
                              >
                              tokens ~=
                              <span class="font-bold"
                                >${(message.usage.total_tokens * token_price).toFixed(6)}</span
                              >
                            </p>
                        </div>
                      </div>
                      {/if} -->
            </div>
          </div>
        </div>
      {/if}
    {/each}
  {/if}
</div>

{#if updating}
  <progress class="progress is-small is-dark" max="100" />
{/if}
<!-- <div class="w-full h-48 bg-red-500">

</div> -->
<div class="flex justify-center md:mb-8">
  <SendMessageForm bind:chatId bind:messageText />
</div>
<!-- <form
  class="field has-addons has-addons-right"
  on:submit|preventDefault={() => submitForm()}
>

  <div class="flex justify-center">
    <div class="" class:is-hidden={!recognition}>
      <button
        class="button is-info is-light"
        class:is-pulse={recording}
        on:click|preventDefault={recordToggle}
        ><span class="greyscale">🎤</span></button
      >
    </div>
    <div class="">
      <button
        class="button is-link is-light"
        on:click|preventDefault={showSettings}
        ><span class="greyscale">⚙️</span></button
      >
    </div>
  </div> -->
<!-- <p class="control is-expanded">
    
    <textarea
      class="input is-info is-focused chat-input"
      placeholder="Type your message here..."
      rows="1"
      
    />
  </p> -->
<!-- <p class="control" class:is-hidden={!recognition}>
    <button
      class="button is-info is-light"
      class:is-pulse={recording}
      on:click|preventDefault={recordToggle}
      ><span class="greyscale">🎤</span></button
    >
  </p> -->
<!-- <p class="control">
    <button
      class="button is-link is-light"
      on:click|preventDefault={showSettings}
      ><span class="greyscale">⚙️</span></button
    >
  </p> -->
<!-- <p class="control">
    <button class="button is-info" type="submit">Send</button>
  </p> -->
<!-- </form> -->

<svelte:window
  on:keydown={(event) => {
    if (event.key === "Escape") {
      closeSettings();
    }
  }}
/>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- <div class="modal" bind:this={settings}>
  <div class="modal-background" on:click={closeSettings} />
  <div class="modal-card">
    <header class="modal-card-head">
      <p class="modal-card-title">Settings</p>
    </header>
    <section class="modal-card-body">
      {#each settingsMap as setting}
        <div class="field is-horizontal">
          <div class="field-label is-normal">
            <label class="label" for="settings-temperature"
              >{setting.name}</label
            >
          </div>
          <div class="field-body">
            <div class="field">
              <input
                class="input"
                type={setting.type}
                id="settings-{setting.key}"
                placeholder={String(setting.default)}
              />
            </div>
          </div>
        </div>
      {/each}
    </section>

    <footer class="modal-card-foot">
      <button class="button is-info" on:click={closeSettings}
        >Close settings</button
      >
      <button class="button" on:click={clearSettings}>Clear settings</button>
    </footer>
  </div>
</div> -->
