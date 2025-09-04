<script lang="ts">
  import PieChart from './PieChart.svelte';
  import { page } from '$app/state';
  import { CHARTS, type ChartKey } from '$lib/stats/aggregations';
  import { type FlightData } from '$lib/utils';

  let {
    flights,
    onOpenChart,
  }: { flights: FlightData[]; onOpenChart?: (key: ChartKey) => void } = $props();

  const user = $derived(page.data.user);
  const ctx = $derived.by(() => ({ userId: user?.id }));

  // Aggregationen (unverändert)
  const seatDistribution = $derived.by(() => CHARTS['seat'].aggregate(flights, ctx));
  const seatClassDistribution = $derived.by(() => CHARTS['seat-class'].aggregate(flights, ctx));
  const reasonDistribution = $derived.by(() => CHARTS['reason'].aggregate(flights, ctx));
  const continentDistribution = $derived.by(() => CHARTS['continents'].aggregate(flights, ctx));
  const topAirlineDistribution = $derived.by(() => CHARTS['airlines'].aggregate(flights, ctx, { limit: 5 }));
  const topAircraftDistribution = $derived.by(() => CHARTS['aircraft-models'].aggregate(flights, ctx, { limit: 5 }));
  const topAircraftRegDistribution = $derived.by(() => CHARTS['aircraft-regs'].aggregate(flights, ctx, { limit: 5 }));
  const topAirportDistribution = $derived.by(() => CHARTS['airports'].aggregate(flights, ctx, { limit: 5 }));
  const topRouteDistribution = $derived.by(() => CHARTS['routes'].aggregate(flights, ctx, { limit: 5 }));

  // UI-Definition der Kacheln (nur Darstellung)
  const cards = $derived.by(() => ([
    { key: 'seat-class',      title: 'Seat Class',           data: seatClassDistribution,   glow: 'rgba(56,189,248,0.20)', at: '15% 10%' },
    { key: 'seat',            title: 'Seat Preference',      data: seatDistribution,        glow: 'rgba(6,182,212,0.20)',  at: '85% 10%' },
    { key: 'reason',          title: 'Flight Reasons',       data: reasonDistribution,      glow: 'rgba(16,185,129,0.20)', at: '20% 90%' },
    { key: 'continents',      title: 'Continents',           data: continentDistribution,   glow: 'rgba(245,158,11,0.18)', at: '100% 100%' },
    { key: 'airlines',        title: 'Top Airlines',         data: topAirlineDistribution,  glow: 'rgba(168,139,250,0.18)',at: '0% 0%' },
    { key: 'aircraft-models', title: 'Top Aircraft Models',  data: topAircraftDistribution, glow: 'rgba(59,130,246,0.18)', at: '100% 0%' },
    { key: 'aircraft-regs',   title: 'Top Specific Aircraft',data: topAircraftRegDistribution, glow: 'rgba(225,29,72,0.18)', at: '0% 100%' },
    { key: 'airports',        title: 'Top Visited Airports', data: topAirportDistribution,  glow: 'rgba(244,63,94,0.18)',  at: '90% 90%' },
    { key: 'routes',          title: 'Top Routes',           data: topRouteDistribution,    glow: 'rgba(20,184,166,0.18)', at: '10% 10%' },
  ]));
</script>

<!-- Kein globaler zusätzlicher Card-Wrapper; nur das Grid -->
<div class="grid md:grid-cols-2 xl:grid-cols-3 gap-4">
  {#each cards as c (c.key)}
    <!-- Gradient-Border-Wrapper (sichtbarer Look-Wechsel) -->
    <div
      class="
        relative rounded-2xl p-[1.5px]
        bg-[conic-gradient(from_140deg,rgba(56,189,248,0.35),rgba(168,85,247,0.30),rgba(244,63,94,0.30),rgba(56,189,248,0.35))]
        transition-transform duration-200 will-change-transform
        hover:scale-[1.01]
        cursor-pointer
      "
      onclick={() => onOpenChart?.(c.key as ChartKey)}
    >
      <!-- Glas-Kachel innen: EIN Rahmen, stärkerer Blur, dunkler Body -->
      <div
        class="
          group relative rounded-2xl
          border border-white/10
          bg-slate-900/70
          backdrop-blur-xl shadow-2xl shadow-black/30
          p-3 md:p-4
        "
      >
        <!-- dezenter Lichthof pro Kachel -->
        <div
          class="pointer-events-none absolute inset-0 -z-10 opacity-50 group-hover:opacity-70 transition-opacity"
          style={`background: radial-gradient(220px 140px at ${c.at}, ${c.glow}, transparent 60%)`}
        />
        <!-- Chart-Fläche -->
        <div class="aspect-square md:aspect-[4/3]">
          <!-- WICHTIG: Chart-Komponente unverändert benutzen -->
          <PieChart data={c.data} title={c.title} />
        </div>
      </div>
    </div>
  {/each}
</div>
