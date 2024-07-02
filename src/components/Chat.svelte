<script lang="ts">
  import {onMount} from "svelte";

  let userMessage = "";
  let history: {role: string; text: string}[] = [];

  let model = null;

  onMount(async () => {
    model = await window.ai.createTextSession();
  });

  function resetHistory() {
    userMessage = "";
    history = [];
  }

  function historyToString() {
    const fh = history.map(({role, text}) => `${role}: ${text}`).join("\n");
    console.log(fh);
    return `
      Użytkownik (USER) prowadzi z tobą (ASSISTANT) przyjazną konwersację. Odpowiadaj na jego zapytania.

      Oto historia rozmowy:
      ${fh}

      Odpowiadaj bez dodatkowego formatowania. Markdown i HTML jest zakazany.
    `;
  }

  function getLastUserMessage() {
    const userMsg = history.filter(({role}) => role === "USER");
    return userMsg[userMsg.length - 1]?.text;
  }

  const submitMessage = async () => {
    history.push({role: "USER", text: userMessage});
    history.push({role: "ASSISTANT", text: ""});
    userMessage = "";

    const stream = await model.promptStreaming(getLastUserMessage());
    for await (const chunk of stream) {
      history[history.length - 1].text = chunk;
    }
  };
</script>

<main>
  {#each history as { role, text }}
    <p class={`message-${role}`}>{text}</p>
  {/each}
  <div class="panel">
    <textarea placeholder="Witam! Jak mogę pomóc? 🤖" bind:value={userMessage}
    ></textarea>
    <div class="buttons">
      <button on:click={submitMessage}>Wyślij</button>
      <button class="reset-btn" on:click={resetHistory}>Reset</button>
    </div>
  </div>
</main>

<style>
  main {
    width: 500px;
    max-width: 500px;
  }

  textarea {
    height: 100px;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .panel {
    display: flex;
    flex-direction: column;
  }

  .panel button {
    margin-top: 10px;
  }

  .message-USER {
    text-align: left;
    background: #533d64;
    border-radius: 5px;
    padding: 4px 10px;
  }

  .message-ASSISTANT {
    text-align: right;
    background: #424652;
    border-radius: 5px;
    padding: 4px 10px;
  }

  .buttons button:nth-child(1) {
    background-color: #3c493e;
  }

  .buttons button:nth-child(2) {
    background-color: #493c42;
  }
</style>
