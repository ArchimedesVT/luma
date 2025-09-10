<script lang="ts">
    import { createEventDispatcher, onMount } from 'svelte';
  
    type Range = { date: string; start: string; end: string };
    type SlotId = `${string}|${string}`; // date|HH:mm
    type SupabaseRow = { user_id?: string | null; date: string; start_time: string; end_time: string; timezone: string };
  
    export let startDate: Date | string;
    export let endDate: Date | string;
    export let dayStart = '09:00';
    export let dayEnd = '17:00';    
    export let stepMinutes = 30;
    export let timezone: string = Intl.DateTimeFormat().resolvedOptions().timeZone;
    export let initialRanges: Range[] = [];
    export let dense = false; // tighter row height
    export let appointmentRanges: Range[] = [];
    export let appointmentText: string[] = [];
  
    const dispatch = createEventDispatcher();
  
    let dates: string[] = []; // YYYY-MM-DD
    let times: string[] = []; // HH:mm across the day window
  
    // Pointer interaction state
    let isDown = false;
    let dragged = false;
    let startCell: SlotId | null = null;
  
    const fmt2 = (n: number) => String(n).padStart(2, '0');
  
    function toISODate(d: Date) {
      const y = d.getFullYear();
      const m = fmt2(d.getMonth() + 1);
      const day = fmt2(d.getDate());
      return `${y}-${m}-${day}`;
    }
  
    function parseDate(input: Date | string): Date {
      if (input instanceof Date) return new Date(input.getFullYear(), input.getMonth(), input.getDate());
      const [y, m, d] = input.split('-').map(Number);
      return new Date(y, (m as number) - 1, d);
    }
  
    function minutesBetween(a: string, b: string) {
      const [ah, am] = a.split(':').map(Number);
      const [bh, bm] = b.split(':').map(Number);
      return (bh * 60 + bm) - (ah * 60 + am);
    }
  
    function addMinutes(t: string, mins: number) {
      const [h, m] = t.split(':').map(Number);
      const total = h * 60 + m + mins;
      return `${fmt2(Math.floor(total / 60))}:${fmt2(total % 60)}`;
    }
  
    function buildTimes() {
      const out: string[] = [];
      const span = minutesBetween(dayStart, dayEnd);
      for (let m = 0; m < span; m += stepMinutes) {
        out.push(addMinutes(dayStart, m));
      }
      return out;
    }
  
    function buildDates() {
      const s = parseDate(startDate);
      const e = parseDate(endDate);
      const out: string[] = [];
      for (let d = new Date(s); d <= e; d.setDate(d.getDate() + 1)) {
        out.push(toISODate(d));
      }
      return out;
    }
  
    function isSlotDisabled(date: string, time: string): boolean {
      return appointmentRanges.some((range, index) => 
        range.date === date && time >= range.start && time < range.end && appointmentText[index] !== undefined
      );
    }
  
    $: times = buildTimes();
    $: dates = buildDates();
    $: if (initialRanges && times.length && dates.length) { /* setFromRanges(initialRanges); */ }
  
    onMount(() => {
      const move = (e: PointerEvent) => {
        if (!isDown) return;
        const el = document.elementFromPoint(e.clientX, e.clientY) as HTMLElement | null;
        if (el && el.dataset && el.dataset.slot) {
          const [d, t] = el.dataset.slot.split('|');
          // applyPaint(d, t); // Removed painting logic
        }
      };
      window.addEventListener('pointermove', move);
      return () => {
        window.removeEventListener('pointermove', move);
      };
    });
  </script>
  
  <style>
    :root { --cell: 36px; --border: #e5e7eb; --muted:#6b7280; --selEdge:#10b981; }
    .grid { overflow:auto; border:1px solid var(--border); border-radius: 14px; user-select: none; }
    .hdr { position: sticky; top: 0; background: white; z-index: 2; }
    .row { display: grid; grid-template-columns: 100px repeat(var(--cols), 1fr); }
    .time { color: var(--muted); border-right: 1px solid var(--border); font-size: 12px; display:flex; align-items:center; justify-content:flex-end; padding-right:8px; }
    .cell { height: var(--cell); border-right: 1px solid var(--border); border-bottom: 1px dotted var(--border); cursor: pointer; position: relative; touch-action: none; display: flex; align-items: center; justify-content: center; } /* Center text in cells */
    .cell.disabled { background: #e5e7eb; cursor: not-allowed; } /* Style for disabled cells */
    .cell:focus { outline: 2px solid var(--selEdge); outline-offset: -2px; }
    .datehdr { padding: 10px; text-align:center; font-weight:600; border-right: 1px solid var(--border); }
    .controls { display:flex; gap:8px; align-items:center; margin-bottom: 8px; flex-wrap: wrap; }
    .pill { border:1px solid var(--border); padding: 6px 10px; border-radius: 999px; font-size: 12px; }
    .btn { padding: 8px 12px; border:1px solid var(--border); border-radius: 10px; background: #111827; color: white; font-weight:600; }
    .btn.ghost { background: white; color: #111827; }
    .legend { display:flex; gap:10px; align-items:center; font-size:12px; color: var(--muted); }
    .swatch { width:14px; height:14px; border-radius:4px; background: var(--selEdge); border: 1px solid var(--selEdge); }
    .more-info { display: none; }
    .show { display: block; }
  </style>
  
  <div class="controls">
    <div class="pill">{dates[0]} → {dates[dates.length - 1]}</div>
    <div class="pill">{dayStart}–{dayEnd} / {stepMinutes}m</div>
    <button class="btn ghost" on:click={() => { /* clearAll logic removed */ }} aria-label="Clear selection">Clear</button>
  </div>
  
  <div class="grid" style={`--cols:${dates.length}`}> 
    <!-- Header row -->
    <div class="row hdr">
      <div class="time" aria-hidden>Time</div>
      {#each dates as d}
        <div class="datehdr">{d}</div>
      {/each}
    </div>
  
    <!-- Time rows -->
    {#each times as t}
      <div class="row">
        <div class="time">{t}</div>
        {#each dates as d}
          {#key `${d}|${t}`}
            <div
              class="cell {isSlotDisabled(d, t) ? 'disabled' : ''}"
              role="button"
              aria-label={`${d} ${t}`}
              tabindex="0"
              data-slot={`${d}|${t}`}
            >
              {#if isSlotDisabled(d, t)}
                {#if appointmentText[appointmentRanges.findIndex(range => range.date === d && t >= range.start && t < range.end)]}
                  {appointmentText[appointmentRanges.findIndex(range => range.date === d && t >= range.start && t < range.end)]}
                  <!-- Removed the button to display more text -->
                  <div class="more-info">
                    {appointmentText[appointmentRanges.findIndex(range => range.date === d && t >= range.start && t < range.end)]}
                  </div>
                {/if}
              {/if}
            </div>
          {/key}
        {/each}
      </div>
    {/each}
  </div>
