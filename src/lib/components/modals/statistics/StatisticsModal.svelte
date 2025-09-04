<script lang="ts">
  import NumberFlow from '@number-flow/svelte';
  import { isBefore } from 'date-fns';

  import ChartDrillDown from './charts/ChartDrillDown.svelte';
  import FlightsPerMonth from './charts/FlightsPerMonth.svelte';
  import FlightsPerWeekday from './charts/FlightsPerWeekday.svelte';
  import PieCharts from './charts/PieCharts.svelte';
  import StatsCard from './StatsCard.svelte';

  import { page } from '$app/state';
  import { Modal } from '$lib/components/ui/modal';
  import { CHARTS, type ChartKey } from '$lib/stats/aggregations';
  import { type FlightData, kmToMiles } from '$lib/utils';
  import { Duration, nowIn } from '$lib/utils/datetime';
  import { round } from '$lib/utils/number';

  let { open = $bindable<boolean>(), allFlights }: { open?: boolean; allFlights: FlightData[] } = $props();

  // Nur abgeschlossene Flüge (unverändert)
  const flights = $derived.by(() =>
    allFlights.filter(
      (f) =>
        !f.date ||
        isBefore(f.arrival ? f.arrival : f.date, nowIn(f.to?.tz || 'UTC')),
    ),
  );

  let isMetric = $derived.by(() => page.data.user?.unit === 'metric');
  let totalDuration = $derived.by(() =>
    Duration.fromSeconds(
      flights.reduce((acc, curr) => (acc += curr.duration ?? 0), 0),
    ),
  );
  let flightCount = $state(0);
  let totalDistance = $state(0);
  let totalDurationParts = $state({ days: 0, hours: 0, minutes: 0 });
  let airports = $state(0);
  let earthCircumnavigations = $state(0);

  // Expanded chart state (unverändert)
  let activeChart: ChartKey | null = $state(null);
  const user = $derived(page.data.user);
  const ctx = $derived.by(() => ({ userId: user?.id }));

  const activeChartData = $derived.by(() => {
    if (!activeChart) return {} as Record<string, number>;
    return CHARTS[activeChart].aggregate(flights, ctx);
  });

  $effect(() => {
    if (open) {
      setTimeout(() => {
        flightCount = flights.length;
        totalDistance = flights.reduce((acc, curr) => (acc += curr.distance ?? 0), 0);
        earthCircumnavigations = totalDistance / 40075;
        const duration = Duration.fromSeconds(
          flights.reduce((acc, curr) => (acc += curr.duration ?? 0), 0),
        );
        totalDurationParts = { days: duration.days, hours: duration.hours, minutes: duration.minutes };
        airports = new Set(
          flights.filter((f) => f.from && f.to).flatMap((f) => [f.from!.name, f.to!.name]),
        ).size;
      }, 200);
    } else {
      flightCount = 0;
      totalDistance = 0;
      totalDurationParts = { days: 0, hours: 0, minutes: 0 };
      airports = 0;
      earthCircumnavigations = 0;
    }
  });
</script>

<svelte:window
  onkeydown={(e) => {
    if (e.key !== 'Escape') return;
    if (activeChart) {
      activeChart = null;
    } else if (open) {
      open = false;
    }
  }}
/>

<Modal
  bind:open
  class="max-w-full h-full overflow-y-auto rounded-none! p-0 md:p-6 bg-gradient-to-br from-slate-950 to-slate-900"
  dialogOnly
  closeOnEscape={false}
>
  {#if activeChart}
    <div class="mx-auto w-full max-w-7xl px-4 md:px-6 py-6 animate-in fade-in-0 zoom-in-95 duration-200">
      <div class="relative overflow-hidden rounded-2xl border border-white/10 bg-slate-900/70 shadow-2xl shadow-black/30 backdrop-blur-xl">
        <div class="absolute inset-0 pointer-events-none bg-[radial-gradient(800px_400px_at_10%_-20%,rgba(56,189,248,0.15),transparent_60%),radial-gradient(700px_400px_at_120%_10%,rgba(244,63,94,0.12),transparent_60%)]" />
        <div class="p-3 md:p-4 flex items-center gap-2 border-b border-white/10">
          <button
            class="inline-flex items-center gap-2 rounded-lg px-3 py-1.5 bg-white/5 hover:bg-white/10 border border-white/10 transition-colors text-sm text-slate-200"
            on:click={() => (activeChart = null)}
          >
            ← Zurück
          </button>
          <span class="text-sm text-slate-300">Detailansicht</span>
        </div>
        <div class="p-4 md:p-6">
          <ChartDrillDown chartKey={activeChart} data={activeChartData} {flights} onBack={() => (activeChart = null)} />
        </div>
      </div>
    </div>
  {:else}
    <div class="mx-auto w-full max-w-7xl px-4 md:px-6 py-6 animate-in fade-in-0 zoom-in-95 duration-200">
      <div class="flex items-center justify-between gap-3">
        <h2 class="text-3xl md:text-4xl font-bold tracking-tight bg-gradient-to-r from-sky-400 via-fuchsia-400 to-rose-400 bg-clip-text text-transparent">
          Statistics
        </h2>
      </div>

      <!-- KPI Grid (nur Aussehen) -->
      <div class="mt-4 grid gap-4 pb-2 sm:grid-cols-2 lg:grid-cols-4">
        <StatsCard class="relative rounded-2xl p-[1.5px] bg-[conic-gradient(from_140deg,rgba(56,189,248,0.35),rgba(168,85,247,0.30),rgba(244,63,94,0.30),rgba(56,189,248,0.35))]">
          <div class="rounded-2xl border border-white/10 bg-slate-900/70 backdrop-blur-xl shadow-xl shadow-black/30 px-6 py-5">
            <h3 class="text-[11px] uppercase tracking-wider text-slate-400">Flights</h3>
            <span class="mt-1 block text-3xl md:text-4xl font-semibold text-slate-100">
              <NumberFlow value={flightCount} />
            </span>
          </div>
        </StatsCard>

        <StatsCard class="relative rounded-2xl p-[1.5px] bg-[conic-gradient(from_140deg,rgba(56,189,248,0.35),rgba(168,85,247,0.30),rgba(244,63,94,0.30),rgba(56,189,248,0.35))]">
          <div class="rounded-2xl border border-white/10 bg-slate-900/70 backdrop-blur-xl shadow-xl shadow-black/30 px-6 py-5">
            <h3 class="text-[11px] uppercase tracking-wider text-slate-400">Distance</h3>
            <span class="mt-1 block text-3xl md:text-4xl font-semibold text-slate-100">
              <NumberFlow
                value={isMetric ? totalDistance : kmToMiles(totalDistance)}
                format={{ style: 'unit', unit: isMetric ? 'kilometer' : 'mile', unitDisplay: 'short', maximumFractionDigits: 0 }}
              />
              <span class="ml-1 text-base text-slate-400">(<NumberFlow value={round(earthCircumnavigations, 2)} />x 🌎)</span>
            </span>
          </div>
        </StatsCard>

        <StatsCard class="relative rounded-2xl p-[1.5px] bg-[conic-gradient(from_140deg,rgba(56,189,248,0.35),rgba(168,85,247,0.30),rgba(244,63,94,0.30),rgba(56,189,248,0.35))]">
          <div class="rounded-2xl border border-white/10 bg-slate-900/70 backdrop-blur-xl shadow-xl shadow-black/30 px-6 py-5">
            <h3 class="text-[11px] uppercase tracking-wider text-slate-400">Duration</h3>
            <span class="mt-1 block text-3xl md:text-4xl font-semibold text-slate-100">
              {#if totalDuration.days}<NumberFlow value={totalDurationParts.days} />d{/if}
              {#if totalDuration.hours}<span class="ml-1"><NumberFlow value={totalDurationParts.hours} />h</span>{/if}
              {#if totalDuration.minutes}<span class="ml-1"><NumberFlow value={totalDurationParts.minutes} />m</span>
              {:else if !totalDuration.days && !totalDuration.hours}<span class="ml-1">0m</span>{/if}
            </span>
          </div>
        </StatsCard>

        <StatsCard class="relative rounded-2xl p-[1.5px] bg-[conic-gradient(from_140deg,rgba(56,189,248,0.35),rgba(168,85,247,0.30),rgba(244,63,94,0.30),rgba(56,189,248,0.35))]">
          <div class="rounded-2xl border border-white/10 bg-slate-900/70 backdrop-blur-xl shadow-xl shadow-black/30 px-6 py-5">
            <h3 class="text-[11px] uppercase tracking-wider text-slate-400">Airports</h3>
            <span class="mt-1 block text-3xl md:text-4xl font-semibold text-slate-100">
              <NumberFlow value={airports} />
            </span>
          </div>
        </StatsCard>
      </div>

      <!-- Nur Abstand um das Grid – kein zweiter Rahmen -->
      <div class="mt-4">
        <PieCharts {flights} onOpenChart={(key) => (activeChart = key)} />
      </div>

      <div class="mt-4 grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="relative rounded-2xl p-[1.5px] bg-[conic-gradient(from_140deg,rgba(56,189,248,0.35),rgba(168,85,247,0.30),rgba(244,63,94,0.30),rgba(56,189,248,0.35))]">
          <div class="rounded-2xl border border-white/10 ">
            <div class="p-4 md:p-6">
              <FlightsPerMonth {flights} />
            </div>
          </div>
        </div>

        <div class="relative rounded-2xl p-[1.5px] bg-[conic-gradient(from_140deg,rgba(56,189,248,0.35),rgba(168,85,247,0.30),rgba(244,63,94,0.30),rgba(56,189,248,0.35))]">
          <div class="rounded-2xl border border-white/10 ">
            <div class="p-4 md:p-6">
              <FlightsPerWeekday {flights} />
            </div>
          </div>
        </div>
      </div>
    </div>
  {/if}
</Modal>
