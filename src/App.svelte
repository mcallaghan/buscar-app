

<script lang="ts">
  import Scatterplot from './Scatterplot.svelte';
  import cdf from '@stdlib/stats-base-dists-hypergeometric-cdf';
  import roundn from '@stdlib/math-base-special-roundn';
  import ns from '@stdlib/array';
  import incrspace from '@stdlib/array/incrspace';
  let relevant: number = 95;
  let consecutive: number = 100;
  let remaining: number = 1000;
  let recall_target: number = 95;
  function kh(r,rt) {
    return Math.floor((r/(rt*0.01))-r+1)
  }
  $: k_hat = kh(relevant, recall_target);
  $: p = roundn(cdf(0, remaining+consecutive, k_hat, consecutive),-3);
  let r_targets = incrspace(0,100,1);
  $: p_scores = r_targets.map((x) => {
    var k_hat_r = kh(relevant,x)
    var p_r = roundn(cdf(0, remaining+consecutive, k_hat_r, consecutive),-3);
    return {
      x: x*0.01,
      y: p_r
    }

  });
  $: fp_scores = p_scores.filter(x => {return x.y>0.01})
  //var
  console.log(fp_scores)

</script>

<main>
<h1> BUSCAR: Biased Urns for efficient Stopping Criteria in technology-Assisted Reviews </h1>
<p>When we use machine learning to assist in screening documents for a systematic
review, we often identify a majority of the relevant records before we have seen every record.
However, if we want to save work, we need to stop early, and we have no way of knowing for sure how
many relevant records we might have missed.
<p>If we see a large number of irrelevant records in a row, this is a good sign that
the proportion of remaining documents that are relevant is low.
<p>Before we use this intuition as a basis for stopping screening, we can calculate
the implications of this estimated low prevalence for recall,
or the proportion of relevant records we actually identify.
<p>The calculator below does this on the basis of the stopping criteria documented in
<a href="https://systematicreviewsjournal.biomedcentral.com/articles/10.1186/s13643-020-01521-4">Callaghan and Müller-Hansen (2020)</a>. Please cite this paper if you use this calculator.
  <div class="parent-container grid">
    <div class="container">
      <Scatterplot points={fp_scores} />
      <p class="caption">
      Plot showing the p value that a range of different recall targets has been missed
    </div>
    <div class="container">
      <p>How many <b>relevant</b> records have you identified?
      <input type="number" bind:value={relevant} min="0" />
      <p>How many <b>consecutive irrelevant</b> records have you identified?
      <input type="number" bind:value={consecutive} min="0"/>
      <p>How many records have <b>not</b> been screened by hand?
      <input type="number" bind:value={remaining} />
      <p>What level of <b>recall</b> (in %) would you like to achieve?
      <input type="number" bind:value={recall_target} min="0" max="100"/>
      <p>
      <p>We would have missed our target if <b>{k_hat}</b> or more documents are relevant.
      <p>The chances of observing <b>0</b> relevant documents in a sample of <b>{consecutive}</b> documents from a total of <b>{consecutive+remaining}</b> documents that contained at least <b>{k_hat}</b> relevant documents is <b>{p}</b>
    </div>
  </div>


</main>

<style>
	.parent-container{

/* 			gap: 1rem; */
	}
	.grid{
			display: grid;
			grid-template-columns: repeat(2, minmax(0,1fr));
	}
	.flex{
			display: flex;
			justify-content: space-between;  /* try out different values here */
	}
  .container {
      display: flex;
    	gap: 1rem;
    	flex-direction: column;
  }
  .caption {
    margin-top: -1em;
    font-size: 0.8em;
  }
</style>
