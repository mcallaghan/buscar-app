

<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import Counter from './lib/Counter.svelte'
  import cdf from '@stdlib/stats-base-dists-hypergeometric-cdf';
  import roundn from '@stdlib/math-base-special-roundn';
  import ns from '@stdlib/array';
  let relevant: number = 95;
  let consecutive: number = 100;
  let remaining: number = 1000;
  let recall_target: number = 95;
  $: k_hat = Math.floor((relevant/(recall_target*0.01))-relevant+1);
  $: p = roundn(cdf(0, remaining+consecutive, k_hat, consecutive),-3);
</script>

<main>
  <div class="card">
  <p>How many relevant documents have you identified?
  <input type="number" bind:value={relevant} min="0" />
  <p>How many consecutive irrelevant documents have you identified?
  <input type="number" bind:value={consecutive} min="0"/>
  <p>How many documents have yet to be screened?
  <input type="number" bind:value={remaining} />
  <p>What level of recall (in %) would you like to achieve?
  <input type="number" bind:value={recall_target} min="0" max="100"/>
  <p>
  <p>We would have missed our target if {k_hat} or more documents are relevant. The chances of observing 0 relevant documents in a sample of {consecutive} documents from a total of {consecutive+remaining} documents that contained at least {k_hat} relevant documents is {p}
</main>

<style>

</style>
