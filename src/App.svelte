<script lang="ts">
  import Scatterplot from './Scatterplot.svelte';
  import { Styles } from '@sveltestrap/sveltestrap';
  import cdf from '@stdlib/stats-base-dists-hypergeometric-cdf';
  import roundn from '@stdlib/math-base-special-roundn';
  import ns from '@stdlib/array';
  import incrspace from '@stdlib/array/incrspace';
  import { Button, Popover, Icon, Input, Label, FormGroup } from '@sveltestrap/sveltestrap';
  $: remaining = d.N.value-d.n_screened.value;
  function kh(r,rt) {
    return Math.floor((r/(rt*0.01))-r+1)
  }
  $: k_hat = kh(d.relevant.value, d.recall_target.value);
  $: n_urn = parseInt(remaining)+parseInt(d.consecutive.value)
  $: p = roundn(cdf(0, n_urn, k_hat, parseInt(d.consecutive.value)),-3);
  let r_targets = incrspace(0,100,1);
  $: p_scores = r_targets.map((x) => {
    var k_hat_r = kh(d.relevant.value,x);
    var n_urn_r = parseInt(remaining)+parseInt(d.consecutive.value);
    var p_r = roundn(cdf(0, n_urn_r, k_hat_r, parseInt(d.consecutive.value)),-3);
    if (x==d.recall_target.value) {
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
  $: fp_scores = p_scores.filter(x => {return x.y>0.005})
  //var
  let d = {
    N: {
      desc: 'How many documents did your queries identify after deduplication?',
      value: 2000,
      step: 10,
      max: null
    },
    n_screened: {
      desc: 'How many have you screened by hand?',
      value: 1000,
      step: 10,
      max: null
    },
    relevant: {
      desc: 'How many relevant records have you identified?',
      value: 95,
      step: 1,
      max: null
    },
    consecutive: {
      desc: 'How many consecutive irrelevant records have you identified?',
      value: 100,
      step: 1,
      max: null
    },
    recall_target: {
      desc: 'What level of recall (in %) would you like to achieve?',
      value: 95,
      step: 1,
      max: null
    }
  }


</script>

<main>
<h1> BUSCAR: Biased Urns for efficient Stopping Criteria in technology-Assisted Reviews </h1>
<p>When we use machine learning to assist in screening documents for a systematic
review, we often identify a majority of the relevant records before we have seen every record.
However, if we want to save work, we need to stop early, and we have no way of knowing for sure how
many relevant records we might have missed.
<p>If we see a large number of irrelevant records in a row, this is a good sign that
the proportion of remaining records that are relevant is low.
<p>Before we use this intuition as a basis for stopping screening, we can consider the number of relevant records we have seen, as well as the number of records we have not yet screened, and  calculate
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

    {#each Object.entries(d) as [e, v]}
    <FormGroup class="w-100" floating label="{v['desc']}">
      <Input
        type="number"
        bind:value={d[e]['value']}
        min="0"
        step={v['step']}
        max={v['max']}
      />
    </FormGroup>
    {/each}
    <p>
    <p>





<p class="border">With p=<b>{p}</b> you have reached your desired recall target
<Icon class="btn" id="p-ex" name="question-diamond" />

<p>This is based on assessing the chances of seeing <b>0</b> relevant records from a sample of <b>{d.consecutive.value}</b> records drawn without replacement from a population of <b>{n_urn}</b> documents that contained at least <b>{k_hat}</b> relevant documents. This is calculated using the <a href="https://github.com/stdlib-js/stats-base-dists-hypergeometric-cdf">hypergeometric distribution function</a>.
<p>In a review, you could state "using the stopping criteria defined in Callaghan and Müller-Hansen (2020), with p={p}, we reject the null hypothesis that we have not achieved our recall target of {d.recall_target.value}%".




    </div>

  </div>
      <p>Note that the hypergeometric distribution assumes that records are retrieved at random. In fact, the use of machine learning <i>generally</i> means we are more likely to retrieve relevant documents than random chance  - this is why we use machine learning prioritisation after all. This means that, as long as machine learning is <b>no worse than random chance</b>, the stopping criterion will be <b>conservative</b>, meaning that if we stop with <b>p=0.05</b> in 100 different reviews, we would have stopped too early in <b>less than</b> 5% of cases. This is good, since we want to minimise the risk of missing relevant studies, but it does mean we often stop later than we could have done. We are currently working on ways to account for this non-random sampling procedure using biased urn theory.

      <p>In this online calculator, we can only calculate a p score based on the number of consecutive irrelevant records. In fact, we can calculate this for any sample of records containing 0 or more relevant records. In our paper, we calculate the score for all possible samples made up of sequences of previously screened records. You can implement this using our <a href="https://mcallaghan.github.io/buscarR/">R package</a> or <a href="https://buscarpy.readthedocs.io/en/latest/">Python package</a>.



      <Popover
        target="p-ex"
        placement="left"
        title="How to interpret p scores">
        The lower your <b>p</b> score, the more confident you can be in stopping.
        If your score is above 0.5, you can't be very confident at all. If it is below 0.5, then it is rather unlikely that you have missed your target (strictly speaking, you would say it is unlikely that X consecutive irrelevant documents would be observed given your null hypothesis that you had missed your recall target). If <b>p</b> is below 0.05, then you can be very confident indeed in rejecting your null hypothesis that the recall target has been missed.
      </Popover>

      <footer class="bd-footer  border-top p-2">
      The Buscar app was built by Max Callaghan: <a href="https://github.com/mcallaghan/buscar-app">Github</a>
      </footer>

</main>

<Styles />

<style>
  .border {
    border: 0.5px solid black;
    padding: 0.5em;
  }
  footer {
    font-size: 0.7em;
  }
  .popover{
    max-width: 600px;
  }
	.parent-container{

/* 			gap: 1rem; */
	}
	.grid{
			display: grid;
			grid-template-columns: repeat(2, minmax(0,1fr));
	}

  .container {
      display: flex;
    	gap: 0.1rem;
    	flex-direction: column;
      padding: 1em;
  }
  .caption {
    margin-top: -1em;
    font-size: 0.8em;
  }
  p {
    text-align:left;
  }
</style>
