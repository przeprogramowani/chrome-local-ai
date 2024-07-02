<script lang="ts">
  import debounce from "debounce";
  import {onMount} from "svelte";
  import {
    SENTIMENT_PROMPT_PL,
    SENTIMENT_PROMPT_EN,
    ECOMMERCE_ASSISTANT_PROMPT_PL,
    ECOMMERCE_ASSISTANT_PROMPT_EN,
  } from "../prompts";

  let comment = "";
  let feedback = "";

  let model = null;

  onMount(async () => {
    model = await window.ai.createTextSession();
  });

  const submitReview = async () => {
    if (!comment || !model) {
      feedback = "";
      return;
    }
    feedback = "...";
    feedback = await model.prompt(ECOMMERCE_ASSISTANT_PROMPT_EN(comment));
  };

  const debouncedSubmitReview = debounce(submitReview, 1000);
</script>

<main>
  <h1>💬 Oceń nasz produkt</h1>
  <textarea
    placeholder="Napisz ocenę produktu"
    bind:value={comment}
    on:input={debouncedSubmitReview}
  ></textarea>
  {#if feedback}
    <p>✨ {feedback}</p>
  {/if}
</main>

<style>
  textarea {
    width: 500px;
    height: 100px;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
</style>
