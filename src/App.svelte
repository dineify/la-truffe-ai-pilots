<script>
  const pages = {
    'reservation-yield': {
      title: 'Reservation Yield + Seating Window Optimizer',
      oneLiner:
        'Fill more prime-time tables with better confirmation timing and smart no-show overbooking guidance.',
      problem:
        'La Truffe Sauvage runs limited dinner windows, so each empty table during peak slots has outsized impact.',
      solution: [
        'Predict no-show risk by reservation time window.',
        'Recommend slot-specific confirmation SMS/call cadence.',
        'Suggest safe overbooking thresholds for prime windows.'
      ],
      inputs: 'Reservation log, table map, arrival times, party sizes, special event flags.',
      outcome: 'Higher covers served per night and fewer empty prime-time tables.',
      pilot: '2-week pilot: nightly recommendations + manager summary before service.',
      cta: 'Approve this if you want immediate revenue lift from existing demand.'
    },
    'kitchen-load': {
      title: 'To-Go vs Dining-Room Kitchen Load Orchestrator',
      oneLiner:
        'Protect in-house dining quality while reducing to-go delays during peak kitchen load.',
      problem:
        'Dinner rush creates contention between in-house tickets and to-go orders, causing delays and stress.',
      solution: [
        'Forecast hour-by-hour kitchen load from reservations and order history.',
        'Recommend dynamic to-go acceptance windows by day.',
        'Generate ready-to-post website/phone messaging for order cutoffs.'
      ],
      inputs: 'To-go timestamps, reservation counts, prep times, staffing level by shift.',
      outcome: 'More on-time service, fewer delayed pickups, smoother line flow.',
      pilot: '2-week pilot: daily go/no-go windows and pre-shift kitchen load brief.',
      cta: 'Approve this if kitchen pacing and service consistency are the top pain today.'
    },
    'occasion-concierge': {
      title: 'Fine-Dining Occasion Concierge',
      oneLiner:
        'Turn anniversaries, birthdays, and gift moments into higher-value repeat bookings.',
      problem:
        'Special-occasion demand exists, but follow-up and personalization are mostly manual and inconsistent.',
      solution: [
        'Detect likely occasion diners from reservation notes and history.',
        'Draft personalized outreach for anniversaries, holidays, and gift certificate moments.',
        'Recommend wine/pairing upsell bundles aligned to party profile.'
      ],
      inputs: 'Reservation notes, guest history, gift certificate activity, seasonal calendar.',
      outcome: 'Higher average check and stronger repeat-visit cadence.',
      pilot: '2-week pilot: weekly opportunity list + ready-to-send outreach drafts.',
      cta: 'Approve this if growth from repeat guests and occasion spend is the main goal.'
    }
  };

  const routeLabel = {
    'reservation-yield': 'Reservation Yield',
    'kitchen-load': 'Kitchen Load',
    'occasion-concierge': 'Occasion Concierge'
  };

  function readRoute() {
    const hash = window.location.hash.replace('#/', '').trim();
    return pages[hash] ? hash : 'reservation-yield';
  }

  let route = readRoute();

  function setRoute(next) {
    route = next;
    window.location.hash = `/${next}`;
  }

  let primeReservations = 42;
  let predictedNoShowRate = 8;
  let safeCaptureRate = 60;
  let overbookShowRate = 80;
  let avgCheck = 95;
  let serviceNightsPerMonth = 12;

  $: predictedNoShows = primeReservations * (predictedNoShowRate / 100);
  $: suggestedOverbook = Math.max(0, Math.floor(predictedNoShows * (safeCaptureRate / 100)));
  $: recoveredCovers = suggestedOverbook * (overbookShowRate / 100);
  $: monthlyRevenueLift = Math.round(recoveredCovers * avgCheck * serviceNightsPerMonth);

  window.addEventListener('hashchange', () => {
    route = readRoute();
  });
</script>

<main class="page">
  <header class="top">
    <p class="eyebrow">La Truffe Sauvage | Lake Charles, Louisiana</p>
    <h1>AI Pilot Concepts</h1>
    <p class="sub">Three concise options to review and pick for a first 2-week pilot.</p>
  </header>

  <div class="print-row">
    <button class="print-btn" on:click={() => window.print()}>Print One-Pager (PDF)</button>
    <p>Print exports only the currently selected pitch.</p>
  </div>

  <nav class="tabs" aria-label="Pitch options">
    {#each Object.keys(routeLabel) as key}
      <button
        class:active={route === key}
        on:click={() => setRoute(key)}
      >
        {routeLabel[key]}
      </button>
    {/each}
  </nav>

  <section class="card">
    <h2>{pages[route].title}</h2>
    <p class="one-liner">{pages[route].oneLiner}</p>

    <div class="grid">
      <article>
        <h3>Problem</h3>
        <p>{pages[route].problem}</p>
      </article>

      <article>
        <h3>What The Agent Does</h3>
        <ul>
          {#each pages[route].solution as item}
            <li>{item}</li>
          {/each}
        </ul>
      </article>

      <article>
        <h3>Inputs Needed</h3>
        <p>{pages[route].inputs}</p>
      </article>

      <article>
        <h3>Expected Outcome</h3>
        <p>{pages[route].outcome}</p>
      </article>

      <article>
        <h3>Pilot Scope</h3>
        <p>{pages[route].pilot}</p>
      </article>

      <article>
        <h3>Decision Trigger</h3>
        <p>{pages[route].cta}</p>
      </article>
    </div>
  </section>

  {#if route === 'reservation-yield'}
    <section class="card prototype">
      <h2>Reservation Yield Pilot Simulator</h2>
      <p class="one-liner">
        Quick estimate for how many extra covers and monthly revenue you can recover from smarter overbooking.
      </p>

      <div class="sim-grid">
        <article>
          <h3>Inputs</h3>
          <div class="control">
            <label for="primeReservations">Prime-time reservations per night</label>
            <input id="primeReservations" type="number" min="1" bind:value={primeReservations} />
          </div>
          <div class="control">
            <label for="predictedNoShowRate">Predicted no-show rate (%)</label>
            <input id="predictedNoShowRate" type="number" min="0" max="50" step="0.5" bind:value={predictedNoShowRate} />
          </div>
          <div class="control">
            <label for="safeCaptureRate">Safe overbook capture rate (%)</label>
            <input id="safeCaptureRate" type="number" min="0" max="100" step="1" bind:value={safeCaptureRate} />
          </div>
          <div class="control">
            <label for="overbookShowRate">Expected show rate for overbooked seats (%)</label>
            <input id="overbookShowRate" type="number" min="0" max="100" step="1" bind:value={overbookShowRate} />
          </div>
          <div class="control">
            <label for="avgCheck">Average check ($)</label>
            <input id="avgCheck" type="number" min="1" step="1" bind:value={avgCheck} />
          </div>
          <div class="control">
            <label for="serviceNightsPerMonth">Service nights per month</label>
            <input id="serviceNightsPerMonth" type="number" min="1" max="31" bind:value={serviceNightsPerMonth} />
          </div>
        </article>

        <article>
          <h3>Projected Nightly Recommendation</h3>
          <ul>
            <li>Predicted no-shows: {predictedNoShows.toFixed(1)} covers</li>
            <li>Suggested safe overbook: {suggestedOverbook} covers</li>
            <li>Expected recovered covers: {recoveredCovers.toFixed(1)}</li>
          </ul>
          <h3 class="result-heading">Projected Monthly Lift</h3>
          <p class="result-value">${monthlyRevenueLift.toLocaleString()}</p>
          <p class="result-note">
            Estimate only. Real pilot tunes by day of week, table mix, and event nights.
          </p>
        </article>
      </div>
    </section>
  {/if}
</main>
