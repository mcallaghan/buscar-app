

<script lang="ts">
  import Scatterplot from './Scatterplot.svelte';
  import cdf from '@stdlib/stats-base-dists-hypergeometric-cdf';
  import roundn from '@stdlib/math-base-special-roundn';
  import ns from '@stdlib/array';
  import incrspace from '@stdlib/array/incrspace';
  let N: number = 2000;
  let n_screened: number = 1000;
  let relevant: number = 95;
  let consecutive: number = 100;
  $: remaining = N-n_screened;
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
    if (x==recall_target) {
      var c = 'c1';
    } else {
      var c = 'c0';
    }
    return {
      x: x*0.01,
      y: p_r,
      c: c
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
      <p>How many documents did your queries identify after deduplication?
      <input type="number" bind:value={N} min="0" step="10"/>
      <p>How many have you screened by hand?
      <input type="number" bind:value={n_screened} min="0" step="10" />
      <p>How many <b>relevant</b> records have you identified?
      <input type="number" bind:value={relevant} min="0" />
      <p>How many <b>consecutive irrelevant</b> records have you identified?
      <input type="number" bind:value={consecutive} min="0"/>
      <p>What level of <b>recall</b> (in %) would you like to achieve?
      <input type="number" bind:value={recall_target} min="0" max="100"/>


<p class="border">With p=<b>{p}</b> you have reached your desired recall target

<p>This is based on assessing the chances of seeing <b>0</b> relevant records from a sample of <b>{consecutive}</b> records drawn from a population of <b>{consecutive+remaining}</b> documents that contained at least <b>{k_hat}</b> relevant documents. This is calculated using the hypergeometric distribution function.
<p>In a review, you could state "using the stopping criteria defined in Callaghan and Müller-Hansen (2020), with p={p}, we reject the null hypothesis that we have not achieved our recall target of {recall_target}%".


    </div>
  </div>


</main>

<style>
  .border {
    border: 0.5px solid black;
  }
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
  input {
    width: 6em;
  }
</style>
