<svelte:options tag="svelte-clock" />

<script>
  import { onMount } from "svelte";
	import { createEventDispatcher } from 'svelte';

  export let clocktitle = "Svelte Clock";
  export let enablems = true;

  let time = new Date();
	const dispatch = createEventDispatcher();
  
  // these automatically update when `time`
  // changes, because of the `$:` prefix
  $: hours = time.getHours();
  $: minutes = time.getMinutes();
  $: seconds = time.getSeconds();
  $: mseconds = time.getMilliseconds();

  onMount(() => {
    const interval = setInterval(updateDate, 10);

    return () => {
      clearInterval(interval);
    };
  });

  function updateDate() {
    time = new Date();
    if (time.getSeconds() % 10 == 0) {
        dispatch('second', {
        text: '10 seconds elapsed!'
      });
    }
  }

  function dispatchSavedDateEvent(e) {
    console.log("[dispatchSecondIsElapsedEvent] time: ", time);
    // 1. Create the custom event.
    const event = new CustomEvent("savedData", {
      detail: time,
      bubbles: true,
      cancelable: true,
      composed: true // makes the event jump shadow DOM boundary
    });

    // 2. Dispatch the custom event.
    this.dispatchEvent(event);
  }
</script>


<h1>{clocktitle}</h1>
<button on:click="{dispatchSavedDateEvent}">Save Date</button>
<svg viewBox="-50 -50 100 100">
  <circle class="clock-face" r="48" />

  <!-- markers -->
  {#each [0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55] as minute}
    <line class="major" y1="35" y2="45" transform="rotate({30 * minute})" />

    {#each [1, 2, 3, 4] as offset}
      <line
        class="minor"
        y1="42"
        y2="45"
        transform="rotate({6 * (minute + offset)})" />
    {/each}
  {/each}

  {#if enablems}
    <!-- ms hand -->
    <g transform="rotate({0.36 * mseconds})">
      <line class="msecond" y1="10" y2="-38" />
      <line class="msecond-counterweight" y1="10" y2="2" />
    </g>
  {/if}

  <!-- second hand -->
  <g transform="rotate({6 * seconds})">
    <line class="second" y1="10" y2="-38" />
    <line class="second-counterweight" y1="10" y2="2" />
  </g>

  <!-- minute hand -->
  <line
    class="minute"
    y1="4"
    y2="-30"
    transform="rotate({6 * minutes + seconds / 10})" />

  <!-- hour hand -->
  <line
    class="hour"
    y1="2"
    y2="-20"
    transform="rotate({30 * hours + minutes / 2})" />

</svg>


<style>
  svg {
    width: 50%;
    height: 50%;
  }

  .clock-face {
    stroke: #333;
    fill: white;
  }

  .minor {
    stroke: #999;
    stroke-width: 0.5;
  }

  .major {
    stroke: #333;
    stroke-width: 1;
  }

  .hour {
    stroke: #333;
  }

  .minute {
    stroke: #666;
  }

  .second,
  .second-counterweight {
    stroke: rgb(180, 0, 0);
  }

  .second-counterweight {
    stroke-width: 3;
  }

  .msecond,
  .msecond-counterweight {
    stroke: rgb(83, 3, 3);
  }

  .msecond-counterweight {
    stroke-width: 1;
  }
</style>