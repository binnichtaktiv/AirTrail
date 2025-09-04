<script lang="ts">
  import { Expand } from '@o7/icon/lucide';
  import { PieChart } from 'layerchart';

  import { cn } from '$lib/utils';

  let {
    title,
    data,
  }: {
    title: string;
    data: Record<string, number>;
  } = $props();

  const noData = $derived.by(
    () => Object.values(data).reduce((a, b) => a + b, 0) === 0,
  );

  const placeholderOrData = $derived.by(() => {
    if (noData) {
      return { 'No data': 1 };
    }
    return Object.fromEntries(
      Object.entries(data)
        .filter(([, value]) => value > 0)
        .sort(([, a], [, b]) => b - a),
    );
  });
</script>

<!-- Außen: Border entfernt -->
<div
  class={cn(
    'relative w-full rounded-sm p-2 h-[250px] border-0', // border-0 statt border-[0.5px] border-zinc-300
    'dark:border-0'                                      // sicherheitshalber auch im Dark-Mode
  )}
>
  <div class="absolute top-4 right-4">
    <Expand size={14} />
  </div>

  <!-- Innen: Verlauf schon transparent gemacht; jetzt Border ebenfalls weg -->
  <div
    class={cn(
      'grid grid-cols-2 items-center h-full rounded-sm',
      'bg-transparent dark:bg-transparent',              // kein Verlauf
      'border-0 dark:border-0 shadow-none'              // keine sichtbare Border/Shadow
    )}
  >
    <div class={cn('h-[160px]')}>
      <PieChart
        data={Object.entries(placeholderOrData).map(([key, value]) => ({
          label: key,
          value,
        }))}
        key="label"
        value="value"
        cRange={noData
          ? ['#3b82f650']
          : ['#3b82f6', '#6366f1', '#8b5cf6', '#a855f7', '#d946ef']}
      />
    </div>

    <div>
      {#if title}
        <div class="flex items-center justify-between pr-2">
          <p><span class="font-bold">{title}</span></p>
        </div>
      {/if}
      {#each Object.entries(data).filter(([, value]) => value > 0) as [key, value] (key)}
        <p><span class="font-bold">{value}</span> {key}</p>
      {/each}
    </div>
  </div>
</div>
