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

  const mvpVersions = {
    v1: {
      title: 'V1 Control Board',
      description: 'Timeline + selected reservation detail + suggested placements.'
    },
    v2: {
      title: 'V2 Floor-First Board',
      description: 'Table occupancy first; assign from unassigned queue.'
    },
    v3: {
      title: 'V3 Action Queue',
      description: 'Prioritized action feed for host decisions in order.'
    }
  };

  const statusMeta = {
    booked: { label: 'Booked', className: 'st-booked' },
    confirmed: { label: 'Confirmed', className: 'st-confirmed' },
    arrived: { label: 'Arrived', className: 'st-arrived' },
    seated: { label: 'Seated', className: 'st-seated' },
    late: { label: 'Late', className: 'st-late' },
    no_show: { label: 'No Show', className: 'st-no-show' },
    cancelled: { label: 'Cancelled', className: 'st-cancelled' },
    walk_in: { label: 'Walk In', className: 'st-walk-in' }
  };

  const serviceConfig = {
    date: '2026-02-19',
    serviceStart: '17:00',
    serviceEnd: '22:00',
    turnMinutes: 110,
    timezone: 'America/Chicago'
  };

  const reservationModelExample = {
    tables: {
      id: 'string',
      label: 'string',
      zone: 'main | patio | bar',
      seats: 'number',
      minParty: 'number',
      mergeGroup: 'string',
      status: 'available | occupied | held | dirty',
      nextAvailableAt: 'HH:mm'
    },
    reservations: {
      id: 'string',
      guestName: 'string',
      phoneLast4: 'string',
      partySize: 'number',
      reservationTime: 'HH:mm',
      status: 'booked | confirmed | arrived | seated | late | no_show | cancelled | walk_in',
      assignedTableId: 'string | null',
      noShowRisk: '0..1',
      source: 'phone | opentable | website | walk_in',
      occasion: 'string',
      notes: 'string',
      createdAt: 'ISO-8601'
    },
    events: {
      id: 'string',
      reservationId: 'string',
      at: 'ISO-8601',
      type: 'status_change | assignment | note',
      actor: 'string',
      payload: 'object'
    }
  };

  const baseTables = [
    { id: 't1', label: 'T1', zone: 'main', seats: 2, minParty: 2, mergeGroup: 'A', status: 'available', nextAvailableAt: '17:00' },
    { id: 't2', label: 'T2', zone: 'main', seats: 2, minParty: 2, mergeGroup: 'A', status: 'available', nextAvailableAt: '17:00' },
    { id: 't3', label: 'T3', zone: 'main', seats: 4, minParty: 2, mergeGroup: 'B', status: 'occupied', nextAvailableAt: '19:05' },
    { id: 't4', label: 'T4', zone: 'main', seats: 4, minParty: 2, mergeGroup: 'B', status: 'available', nextAvailableAt: '17:00' },
    { id: 't5', label: 'T5', zone: 'patio', seats: 4, minParty: 2, mergeGroup: 'C', status: 'held', nextAvailableAt: '18:45' },
    { id: 't6', label: 'T6', zone: 'patio', seats: 6, minParty: 3, mergeGroup: 'C', status: 'available', nextAvailableAt: '17:00' },
    { id: 't7', label: 'T7', zone: 'bar', seats: 2, minParty: 1, mergeGroup: 'D', status: 'available', nextAvailableAt: '17:00' },
    { id: 't8', label: 'T8', zone: 'main', seats: 8, minParty: 5, mergeGroup: 'E', status: 'dirty', nextAvailableAt: '17:35' }
  ];

  const baseReservations = [
    {
      id: 'r-201',
      guestName: 'J. Martin',
      phoneLast4: '0183',
      partySize: 2,
      reservationTime: '17:30',
      status: 'confirmed',
      assignedTableId: 't1',
      noShowRisk: 0.09,
      source: 'phone',
      occasion: 'Anniversary',
      notes: 'Prefers quiet table',
      createdAt: '2026-02-18T13:01:00-06:00'
    },
    {
      id: 'r-202',
      guestName: 'L. Broussard',
      phoneLast4: '4377',
      partySize: 4,
      reservationTime: '18:00',
      status: 'booked',
      assignedTableId: null,
      noShowRisk: 0.41,
      source: 'website',
      occasion: 'Birthday',
      notes: 'Requested window if available',
      createdAt: '2026-02-18T16:42:00-06:00'
    },
    {
      id: 'r-203',
      guestName: 'S. LeBlanc',
      phoneLast4: '9321',
      partySize: 6,
      reservationTime: '18:30',
      status: 'confirmed',
      assignedTableId: 't6',
      noShowRisk: 0.18,
      source: 'opentable',
      occasion: 'Business',
      notes: 'Needs prompt seating by 6:35',
      createdAt: '2026-02-17T18:00:00-06:00'
    },
    {
      id: 'r-204',
      guestName: 'A. Fontenot',
      phoneLast4: '2239',
      partySize: 2,
      reservationTime: '19:00',
      status: 'booked',
      assignedTableId: null,
      noShowRisk: 0.52,
      source: 'website',
      occasion: 'Date Night',
      notes: 'No notes',
      createdAt: '2026-02-19T09:33:00-06:00'
    },
    {
      id: 'r-205',
      guestName: 'P. Nguyen',
      phoneLast4: '6671',
      partySize: 8,
      reservationTime: '19:30',
      status: 'confirmed',
      assignedTableId: 't8',
      noShowRisk: 0.22,
      source: 'phone',
      occasion: 'Family Dinner',
      notes: 'High-chair requested',
      createdAt: '2026-02-15T14:06:00-06:00'
    },
    {
      id: 'r-206',
      guestName: 'Walk-in Queue',
      phoneLast4: '----',
      partySize: 2,
      reservationTime: '20:00',
      status: 'walk_in',
      assignedTableId: null,
      noShowRisk: 0,
      source: 'walk_in',
      occasion: 'N/A',
      notes: 'Added by host stand',
      createdAt: '2026-02-19T18:01:00-06:00'
    }
  ];

  const baseEvents = [
    {
      id: 'e-001',
      reservationId: 'r-202',
      at: '2026-02-19T14:42:00-06:00',
      type: 'status_change',
      actor: 'nightly-agent',
      payload: { from: 'booked', to: 'booked', riskFlag: 'high_no_show' }
    },
    {
      id: 'e-002',
      reservationId: 'r-203',
      at: '2026-02-19T16:11:00-06:00',
      type: 'assignment',
      actor: 'host',
      payload: { tableId: 't6' }
    },
    {
      id: 'e-003',
      reservationId: 'r-201',
      at: '2026-02-19T17:28:00-06:00',
      type: 'status_change',
      actor: 'host',
      payload: { from: 'confirmed', to: 'arrived' }
    }
  ];

  function readRoute() {
    const hash = window.location.hash.replace('#/', '').trim();
    return pages[hash] ? hash : 'reservation-yield';
  }

  let route = readRoute();

  function setRoute(next) {
    route = next;
    window.location.hash = `/${next}`;
  }

  let tables = structuredClone(baseTables);
  let reservations = structuredClone(baseReservations);
  let events = structuredClone(baseEvents);
  let selectedReservationId = reservations[0]?.id ?? null;

  let selectedMvpVersion = 'v1';
  let shiftMode = false;

  let walkInName = 'Walk-in Guest';
  let walkInPartySize = 2;

  let primeReservations = 42;
  let predictedNoShowRate = 8;
  let safeCaptureRate = 60;
  let overbookShowRate = 80;
  let avgCheck = 95;
  let serviceNightsPerMonth = 12;

  $: selectedReservation = reservations.find((r) => r.id === selectedReservationId) ?? null;
  $: sortedReservations = [...reservations].sort((a, b) => a.reservationTime.localeCompare(b.reservationTime));
  $: activeReservations = sortedReservations.filter((r) => ['booked', 'confirmed', 'arrived', 'seated', 'walk_in', 'late'].includes(r.status));
  $: atRiskReservations = sortedReservations.filter(
    (r) => (r.status === 'booked' || r.status === 'confirmed') && r.noShowRisk >= 0.35
  );
  $: upcomingUnassigned = sortedReservations.filter(
    (r) => (r.status === 'booked' || r.status === 'confirmed' || r.status === 'walk_in' || r.status === 'arrived') && !r.assignedTableId
  );

  $: predictedNoShows = primeReservations * (predictedNoShowRate / 100);
  $: suggestedOverbook = Math.max(0, Math.floor(predictedNoShows * (safeCaptureRate / 100)));
  $: recoveredCovers = suggestedOverbook * (overbookShowRate / 100);
  $: monthlyRevenueLift = Math.round(recoveredCovers * avgCheck * serviceNightsPerMonth);

  $: timelineSlots = buildTimeline(serviceConfig.serviceStart, serviceConfig.serviceEnd, 30);
  $: slotLoad = timelineSlots.map((slot) => {
    const covers = reservations
      .filter((r) => ['booked', 'confirmed', 'arrived', 'seated', 'walk_in'].includes(r.status))
      .filter((r) => isWithinWindow(slot, r.reservationTime, serviceConfig.turnMinutes))
      .reduce((sum, r) => sum + r.partySize, 0);
    return { slot, covers };
  });

  $: totalExpectedCovers = reservations
    .filter((r) => ['booked', 'confirmed', 'arrived', 'seated', 'walk_in'].includes(r.status))
    .reduce((sum, r) => sum + r.partySize, 0);

  $: noShowExposure = reservations
    .filter((r) => r.status === 'booked' || r.status === 'confirmed')
    .reduce((sum, r) => sum + Math.round(r.partySize * r.noShowRisk), 0);

  $: recommendedCallList = atRiskReservations.slice(0, 4);
  $: suggestedTables = selectedReservation ? suggestTables(selectedReservation, tables, reservations) : [];
  $: tablesByZone = ['main', 'patio', 'bar'].map((zone) => ({
    zone,
    tables: tables.filter((t) => t.zone === zone)
  }));
  $: arrivalQueue = activeReservations.filter((r) => !r.assignedTableId && (r.status === 'arrived' || r.status === 'walk_in'));

  $: actionQueue = buildActionQueue(reservations);

  function parseMinutes(hhmm) {
    const [h, m] = hhmm.split(':').map(Number);
    return h * 60 + m;
  }

  function isWithinWindow(slot, reservationTime, turnMinutes) {
    const slotMin = parseMinutes(slot);
    const start = parseMinutes(reservationTime);
    return slotMin >= start && slotMin < start + turnMinutes;
  }

  function buildTimeline(start, end, interval) {
    const slots = [];
    const startMin = parseMinutes(start);
    const endMin = parseMinutes(end);
    for (let t = startMin; t <= endMin; t += interval) {
      const hh = String(Math.floor(t / 60)).padStart(2, '0');
      const mm = String(t % 60).padStart(2, '0');
      slots.push(`${hh}:${mm}`);
    }
    return slots;
  }

  function suggestionConflict(a, b) {
    return (
      isWithinWindow(a.reservationTime, b.reservationTime, serviceConfig.turnMinutes) ||
      isWithinWindow(b.reservationTime, a.reservationTime, serviceConfig.turnMinutes)
    );
  }

  function suggestTables(reservation, allTables, allReservations) {
    const usedTableIds = new Set(
      allReservations
        .filter((r) => r.id !== reservation.id)
        .filter((r) => ['booked', 'confirmed', 'arrived', 'seated', 'walk_in', 'late'].includes(r.status))
        .filter((r) => suggestionConflict(r, reservation))
        .map((r) => r.assignedTableId)
        .filter(Boolean)
    );

    return allTables
      .filter((t) => t.seats >= reservation.partySize)
      .filter((t) => t.minParty <= reservation.partySize)
      .map((t) => {
        const seatWaste = t.seats - reservation.partySize;
        const conflict = usedTableIds.has(t.id) ? 1 : 0;
        const statusPenalty = t.status === 'available' ? 0 : t.status === 'held' ? 1 : 2;
        const score = seatWaste + conflict * 6 + statusPenalty * 2;
        return { ...t, score, usedSoon: usedTableIds.has(t.id) };
      })
      .sort((a, b) => a.score - b.score)
      .slice(0, 4);
  }

  function findReservationOnTable(tableId) {
    return activeReservations.find((r) => r.assignedTableId === tableId && r.status !== 'no_show' && r.status !== 'cancelled');
  }

  function assignTable(reservationId, tableId) {
    reservations = reservations.map((r) =>
      r.id === reservationId ? { ...r, assignedTableId: tableId } : r
    );
    events = [
      {
        id: `e-${Date.now()}`,
        reservationId,
        at: new Date().toISOString(),
        type: 'assignment',
        actor: 'host',
        payload: { tableId }
      },
      ...events
    ];
  }

  function assignBestTable(reservationId) {
    const reservation = reservations.find((r) => r.id === reservationId);
    if (!reservation) return;
    const best = suggestTables(reservation, tables, reservations)[0];
    if (best) assignTable(reservationId, best.id);
  }

  function updateReservationStatus(reservationId, nextStatus) {
    const current = reservations.find((r) => r.id === reservationId);
    if (!current) return;

    reservations = reservations.map((r) =>
      r.id === reservationId ? { ...r, status: nextStatus } : r
    );

    events = [
      {
        id: `e-${Date.now()}`,
        reservationId,
        at: new Date().toISOString(),
        type: 'status_change',
        actor: 'host',
        payload: { from: current.status, to: nextStatus }
      },
      ...events
    ];

    if (nextStatus === 'seated' && current.assignedTableId) {
      tables = tables.map((t) =>
        t.id === current.assignedTableId
          ? { ...t, status: 'occupied', nextAvailableAt: bumpTime(current.reservationTime, serviceConfig.turnMinutes) }
          : t
      );
    }
  }

  function bumpTime(hhmm, mins) {
    const value = parseMinutes(hhmm) + mins;
    const hh = String(Math.floor(value / 60)).padStart(2, '0');
    const mm = String(value % 60).padStart(2, '0');
    return `${hh}:${mm}`;
  }

  function addWalkIn() {
    const now = new Date();
    const localTime = `${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`;
    const newId = `r-${Math.floor(Math.random() * 900 + 100)}`;
    const newWalkIn = {
      id: newId,
      guestName: walkInName,
      phoneLast4: '----',
      partySize: Math.max(1, Number(walkInPartySize) || 2),
      reservationTime: localTime,
      status: 'walk_in',
      assignedTableId: null,
      noShowRisk: 0,
      source: 'walk_in',
      occasion: 'N/A',
      notes: 'Added at host stand',
      createdAt: now.toISOString()
    };

    reservations = [...reservations, newWalkIn];
    selectedReservationId = newId;
    walkInName = 'Walk-in Guest';
    walkInPartySize = 2;
  }

  function buildActionQueue(allReservations) {
    const queue = [];
    allReservations.forEach((r) => {
      if ((r.status === 'booked' || r.status === 'confirmed') && r.noShowRisk >= 0.45) {
        queue.push({
          id: `risk-${r.id}`,
          priority: 1,
          label: 'Call to confirm high-risk no-show',
          reservationId: r.id,
          reservation: r,
          action: 'confirm_call'
        });
      }

      if ((r.status === 'arrived' || r.status === 'walk_in') && !r.assignedTableId) {
        queue.push({
          id: `seat-${r.id}`,
          priority: 0,
          label: 'Seat now: guest already at host stand',
          reservationId: r.id,
          reservation: r,
          action: 'assign_table'
        });
      }

      if ((r.status === 'booked' || r.status === 'confirmed') && !r.assignedTableId) {
        queue.push({
          id: `assign-${r.id}`,
          priority: 2,
          label: 'Pre-assign table before rush',
          reservationId: r.id,
          reservation: r,
          action: 'assign_table'
        });
      }
    });

    return queue.sort((a, b) => a.priority - b.priority || a.reservation.reservationTime.localeCompare(b.reservation.reservationTime));
  }

  function markActionDone(item) {
    if (item.action === 'confirm_call') {
      updateReservationStatus(item.reservationId, 'confirmed');
      return;
    }
    assignBestTable(item.reservationId);
  }

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
      <button class:active={route === key} on:click={() => setRoute(key)}>
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
      <h2>MVP Data Model (v1)</h2>
      <p class="one-liner">
        Structured to support both nightly risk scoring and same-day host stand updates.
      </p>
      <pre>{JSON.stringify(reservationModelExample, null, 2)}</pre>
    </section>

    <section class={`card prototype reservation-mvp ${shiftMode ? 'shift' : ''}`}>
      <div class="mvp-top-row">
        <div>
          <h2>Reservation Board MVP Variants</h2>
          <p class="one-liner">Toggle versions and test which workflow staff prefers in real service.</p>
        </div>
        <button class={`mode-toggle ${shiftMode ? 'on' : ''}`} on:click={() => (shiftMode = !shiftMode)}>
          Shift Mode: {shiftMode ? 'ON' : 'OFF'}
        </button>
      </div>

      <div class="version-tabs">
        {#each Object.keys(mvpVersions) as key}
          <button class:active={selectedMvpVersion === key} on:click={() => (selectedMvpVersion = key)}>
            {mvpVersions[key].title}
          </button>
        {/each}
      </div>
      <p class="version-sub">{mvpVersions[selectedMvpVersion].description}</p>

      <div class="board-metrics">
        <article>
          <h3>Expected Covers</h3>
          <p class="metric">{totalExpectedCovers}</p>
        </article>
        <article>
          <h3>No-Show Exposure</h3>
          <p class="metric">{noShowExposure} covers</p>
        </article>
        <article>
          <h3>High-Risk Calls</h3>
          <p class="metric">{recommendedCallList.length}</p>
        </article>
        <article>
          <h3>Unassigned</h3>
          <p class="metric">{upcomingUnassigned.length}</p>
        </article>
      </div>

      {#if selectedMvpVersion === 'v1'}
        {#if shiftMode}
          <div class="shift-note">Shift mode: bigger touch targets, minimal context, fastest actions.</div>
        {/if}

        <div class="board-grid">
          <article>
            <h3>Reservations Timeline</h3>
            <div class="res-list">
              {#each sortedReservations as res}
                <button
                  class="res-item"
                  class:selected={selectedReservationId === res.id}
                  on:click={() => (selectedReservationId = res.id)}
                >
                  <div>
                    <p class="res-time">{res.reservationTime}</p>
                    <p class="res-name">{res.guestName} ({res.partySize})</p>
                  </div>
                  <div class="res-meta">
                    <span class={`status-pill ${statusMeta[res.status]?.className || ''}`}>
                      {statusMeta[res.status]?.label || res.status}
                    </span>
                    <span class="risk">Risk {(res.noShowRisk * 100).toFixed(0)}%</span>
                  </div>
                </button>
              {/each}
            </div>

            <h3 class="section-gap">Add Walk-In</h3>
            <div class="control">
              <label for="walkInName">Guest / label</label>
              <input id="walkInName" bind:value={walkInName} />
            </div>
            <div class="control">
              <label for="walkInPartySize">Party size</label>
              <input id="walkInPartySize" type="number" min="1" max="12" bind:value={walkInPartySize} />
            </div>
            <button class="action-btn" on:click={addWalkIn}>Add Walk-In Now</button>
          </article>

          <article>
            {#if selectedReservation}
              <h3>Selected Reservation</h3>
              <p class="selected-heading">
                {selectedReservation.guestName} at {selectedReservation.reservationTime} (party {selectedReservation.partySize})
              </p>
              {#if !shiftMode}
                <p class="selected-detail">
                  Source: {selectedReservation.source} | Occasion: {selectedReservation.occasion}
                </p>
                <p class="selected-detail">Notes: {selectedReservation.notes}</p>
              {/if}
              <p class="selected-detail">Current table: {selectedReservation.assignedTableId || 'Unassigned'}</p>

              <h3 class="section-gap">Quick Status Actions</h3>
              <div class="status-actions">
                <button on:click={() => updateReservationStatus(selectedReservation.id, 'arrived')}>Arrived</button>
                <button on:click={() => updateReservationStatus(selectedReservation.id, 'seated')}>Seated</button>
                <button on:click={() => updateReservationStatus(selectedReservation.id, 'late')}>Late</button>
                <button on:click={() => updateReservationStatus(selectedReservation.id, 'no_show')}>No Show</button>
                <button on:click={() => updateReservationStatus(selectedReservation.id, 'cancelled')}>Cancel</button>
              </div>

              <h3 class="section-gap">Suggested Table Placements</h3>
              <div class="table-suggest-list">
                {#each suggestedTables as t}
                  <div class="table-suggest">
                    <div>
                      <p class="table-line">{t.label} | {t.zone} | {t.seats} seats</p>
                      <p class="table-sub">
                        {t.status}, next {t.nextAvailableAt}{#if t.usedSoon} (conflict risk){/if}
                      </p>
                    </div>
                    <button class="action-btn slim" on:click={() => assignTable(selectedReservation.id, t.id)}>
                      Assign
                    </button>
                  </div>
                {/each}
              </div>
            {:else}
              <p>Select a reservation to start.</p>
            {/if}
          </article>
        </div>
      {/if}

      {#if selectedMvpVersion === 'v2'}
        {#if shiftMode}
          <div class="shift-note">Shift mode: table cards + one-tap seat from queue.</div>
        {/if}

        <div class="board-grid secondary">
          <article>
            <h3>Unassigned / Arrivals Queue</h3>
            <div class="queue-list">
              {#each upcomingUnassigned as res}
                <div class="queue-item">
                  <div>
                    <p class="queue-title">{res.reservationTime} - {res.guestName} ({res.partySize})</p>
                    <p class="queue-sub">{statusMeta[res.status]?.label} | risk {(res.noShowRisk * 100).toFixed(0)}%</p>
                  </div>
                  <div class="queue-actions">
                    <button class="action-btn slim" on:click={() => assignBestTable(res.id)}>Assign Best</button>
                    <button class="action-btn slim" on:click={() => (selectedReservationId = res.id)}>Open</button>
                  </div>
                </div>
              {/each}
            </div>
          </article>

          <article>
            <h3>Floor Overview by Zone</h3>
            {#each tablesByZone as zoneGroup}
              <h4>{zoneGroup.zone}</h4>
              <div class="table-cards">
                {#each zoneGroup.tables as t}
                  {@const assigned = findReservationOnTable(t.id)}
                  <div class={`table-card ${t.status}`}>
                    <p class="table-line">{t.label} ({t.seats})</p>
                    <p class="table-sub">{t.status} | next {t.nextAvailableAt}</p>
                    {#if assigned}
                      <p class="queue-title">{assigned.guestName} ({assigned.partySize})</p>
                      <p class="queue-sub">{assigned.reservationTime} | {statusMeta[assigned.status]?.label}</p>
                    {:else}
                      <p class="queue-sub">No assignment</p>
                    {/if}
                  </div>
                {/each}
              </div>
            {/each}
          </article>
        </div>
      {/if}

      {#if selectedMvpVersion === 'v3'}
        {#if shiftMode}
          <div class="shift-note">Shift mode: action queue only, no deep drill-down.</div>
        {/if}

        <div class="board-grid secondary">
          <article>
            <h3>Priority Action Queue</h3>
            <div class="queue-list">
              {#each actionQueue as item}
                <div class={`queue-item prio-${item.priority}`}>
                  <div>
                    <p class="queue-title">{item.label}</p>
                    <p class="queue-sub">
                      {item.reservation.reservationTime} - {item.reservation.guestName} ({item.reservation.partySize})
                    </p>
                  </div>
                  <div class="queue-actions">
                    <button class="action-btn slim" on:click={() => markActionDone(item)}>
                      {item.action === 'confirm_call' ? 'Mark Called' : 'Assign Best'}
                    </button>
                    {#if !shiftMode}
                      <button class="action-btn slim" on:click={() => (selectedReservationId = item.reservationId)}>
                        Open
                      </button>
                    {/if}
                  </div>
                </div>
              {/each}
            </div>
          </article>

          <article>
            <h3>Service Load Curve</h3>
            <div class="slot-grid">
              {#each slotLoad as slot}
                <div class="slot-chip" class:hot={slot.covers >= 18}>
                  <span>{slot.slot}</span>
                  <strong>{slot.covers}</strong>
                </div>
              {/each}
            </div>

            <h3 class="section-gap">Nightly Call List</h3>
            <ul>
              {#each recommendedCallList as res}
                <li>{res.reservationTime} - {res.guestName} ({res.partySize}) risk {(res.noShowRisk * 100).toFixed(0)}%</li>
              {/each}
            </ul>
          </article>
        </div>
      {/if}
    </section>

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
