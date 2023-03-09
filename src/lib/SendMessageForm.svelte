<!-- SendMessageForm.svelte -->
<script lang="ts">
  export let messageText = "";
  import Icon from "@iconify/svelte";
  import { afterUpdate, onMount } from "svelte";
  import {
    apiKeyStorage,
    chatsStorage,
    addMessage,
    clearMessages,
  } from "./Storage.svelte";
  import type { Request, Response, Message, Settings } from "./Types.svelte";

  export let chatId: number;
  let updating: boolean = false;

  function sendMessage() {
    if (messageText.trim()) {
      console.log(`Sending message: ${messageText}`);
      messageText = "";
    }
  }

  $: chat = $chatsStorage.find((chat) => chat.id === chatId);
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

  let input: HTMLTextAreaElement;
  let recognition: any = null;
  let recording = false;
  // Focus the input on mount
  onMount(() => {
    // input.focus();

    // Try to detect speech recognition support
    if ("SpeechRecognition" in window) {
      // @ts-ignore
      recognition = new SpeechRecognition();
    } else if ("webkitSpeechRecognition" in window) {
      // @ts-ignore
      recognition = new webkitSpeechRecognition();
    }

    if (recognition) {
      recognition.interimResults = false;
      recognition.onstart = () => {
        recording = true;
      };
      recognition.onresult = (event) => {
        // Stop speech recognition, submit the form and remove the pulse
        const last = event.results.length - 1;
        const text = event.results[last][0].transcript;
        messageText = text;
        recognition.stop();
        recording = false;
        // submitForm(true);
      };
    } else {
      console.log("Speech recognition not supported");
    }
  });

  const recordToggle = () => {
    // Check if already recording - if so, stop - else start
    if (recording) {
      recognition?.stop();
      recording = false;
    } else {
      recognition?.start();
    }
  };

  function inputAutogrow(event) {
    const input = event.target;
    input.style.width = "auto";
    input.style.width = input.scrollWidth + "px";
    console.log(input.style.width);
  }

  // Autogrow textareas
  function textareaAutogrow(event) {
    const textarea = event.target;
    // textarea.style.height = 'auto';
    // textarea.style.height = textarea.scrollHeight + 'px';
    textarea.style.height = calcHeight(textarea.value) + "px";
  }

  // Dealing with Textarea Height
  // https://css-tricks.com/auto-growing-inputs-textareas/
  function calcHeight(value) {
    let numberOfLineBreaks = (value.match(/\n/g) || []).length;
    // min-height + lines x line-height + padding + border
    let newHeight = 24 + numberOfLineBreaks * 24;
    return newHeight;
  }

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

  const clearSettings = () => {
    console.log(settings);
    settingsMap.forEach((setting) => {
      const input = settings.querySelector(
        `#settings-${setting.key}`
      ) as HTMLInputElement;
      input.value = "";
    });
  };
  const showSettings = () => {
    console.log(settingsMap);
  };
  $: isEmpty = messageText === "";
</script>

<!-- border-gray-200 border -->
<!-- m-0 w-full resize-none border-0 bg-transparent p-0 pl-2 pr-7 focus:ring-0 focus-visible:ring-0 dark:bg-transparent md:pl-0 -->
<div class="p-2 flex bg-slate-700 w-full md:w-2/3">
  <div>
    <div class="dropdown dropdown-top">
      <label tabindex="0" class="btn btn-ghost text-white"
        ><Icon
          icon="mdi:gear"
          class="text-xl hover:rotate-45 transition ease-in-out"
        /></label
      >
      <ul
        tabindex="0"
        class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-52"
      >
        <li>
          <a
            href={"#"}
            on:click={() => {
              clearMessages(chatId);
            }}><Icon icon="mdi:broom" class="text-xl" />Clean All</a
          >
        </li>
        <li>
          <a href={"#"}
            ><Icon icon="icon-park:setting-config" class="text-xl" /><label
              for="modal-settings"
              class="">Setting</label
            ></a
          >
        </li>
        <li>
          <!-- on:click|preventDefault={suggestName()} -->
          <a href={"#"} 
            ><Icon icon="ic:outline-smart-toy" class="text-xl" />Suggest Name</a
          >
        </li>
      </ul>
      <!-- icon-park:setting-config -->
    </div>
  </div>
  <form
    class="relative w-full py-1 rounded border border-white text-white"
    on:submit|preventDefault={() => submitForm()}
  >
    <textarea
      class="m-0 w-full textarea leading-5 max-h-36 min-h-0  bg-transparent resize-none border-0 flex-grow rounded-md pl-2 pr-7"
      placeholder="Type a message"
      rows="1"
      on:keydown={(e) => {
        // Only send if Enter is pressed, not Shift+Enter
        if (e.key === "Enter" && !e.shiftKey) {
          // submitForm();
          e.preventDefault();
        }
      }}
      on:input={(e) => {
        // Resize the textarea to fit the content - auto is important to reset the height after deleting content
        input.style.height = "auto";
        input.style.height = input.scrollHeight + "px";
      }}
      bind:this={input}
      bind:value={messageText}
    />
    <!-- on:input={textareaAutogrow} -->
    {#if isEmpty}
      <button
        class="btn  btn-ghost rounded-md absolute p-1 bottom-1 right-1"
        on:click|preventDefault={recordToggle}
      >
        {#if recording}
          <Icon
            icon="material-symbols:stop-circle-outline"
            class="text-error md:text-2xl"
          />
          <span
            class="animate-ping absolute inline-flex h-5 w-5 rounded-full bg-red-400 opacity-75"
          />
        {:else}
          <Icon icon="carbon:microphone" class="text-base md:text-2xl" />
        {/if}
      </button>
      <!-- <div class="" class:is-hidden={!recognition}>
        <button
          class="btn"
          class:animate-ping={recording}
          on:click|preventDefault={recordToggle}
          ><span class="greyscale">🎤</span></button
        >
      </div> -->
    {:else}
      <button class="btn btn-ghost rounded-md absolute p-1 bottom-1 right-1"
        ><Icon icon="eva:paper-plane-fill" class="text-base md:text-2xl" />
      </button>
      <!-- on:click={submitForm()} -->
    {/if}
  </form>

  <!-- <Icon icon="carbon:microphone" width="26" /> -->
  <!-- <button class="btn">
  </button> -->
  <!-- <input type="text" class="flex-grow rounded-md border-gray-200 border p-2 mr-2" placeholder="Type a message" bind:value={messageText} on:input={inputAutogrow}
  style="width: 100%; box-sizing: border-box;"> -->
  <!-- overflow-y-hidden -->
</div>
<svelte:window
  on:keydown={(event) => {
    if (event.key === "Escape") {
      // closeSettings();
    }
  }}
/>
<input type="checkbox" id="modal-settings" class="hidden modal-toggle" />
<div class="modal modal-bottom sm:modal-middle">
  <div class="modal-box" bind:this={settings}>
    <h3 class="font-bold text-lg">Settings</h3>
    {#each settingsMap as setting}
      <div class="field is-horizontal">
        <div class="field-label is-normal">
          <label class="label" for={setting.name}>{setting.name}</label>
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
    <div class="modal-action">
      <label for="modal-settings" class="btn btn-info">Close settings</label>
      <label class="btn" on:click={clearSettings}>Clear settings</label>
    </div>
  </div>
</div>

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
            <label class="label" for="settings-temperature">{setting.name}</label>
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
  </div>
</div> -->
<style>
  .textarea:focus {
    outline: none !important;
  }
</style>
