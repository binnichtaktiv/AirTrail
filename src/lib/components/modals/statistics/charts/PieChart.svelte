<script lang="ts">
  import { Expand } from '@o7/icon/lucide';
  import { pie, arc, type PieArcDatum } from 'd3-shape';
  import { cn } from '$lib/utils';

  let {
    title,
    data,
  }: {
    title: string;
    data: Record<string, number>;
  } = $props();

  const SIZE = 160;
  const RADIUS = SIZE / 2 - 2;

  const COLORS = [
    '#06B6D4',
    '#8B5CF6',
    '#EC4899',
    '#F97316',
    '#FACC15',
    '#1DB954',
  ];

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
        .filter(([key]) => key !== 'No Data')
        .sort(([, a], [, b]) => b - a),
    );
  });

  const pieData = $derived.by(() =>
    pie<{ label: string; value: number }>().value((d) => d.value)(
      Object.entries(placeholderOrData).map(([label, value]) => ({
        label,
        value,
      })),
    ),
  );

  const arcGen = arc<PieArcDatum<{ label: string; value: number }>>()
    .innerRadius(0)
    .outerRadius(RADIUS);
</script>

<div
  class={cn(
    'relative w-full border-[0.5px] border-zinc-300 rounded-sm p-2 dark:border-zinc-800 h-[250px]',
  )}
>
  <div class="absolute top-4 right-4">
    <Expand size={14} />
  </div>

  <div
    class={cn(
      'grid grid-cols-2 items-center border rounded-sm h-full',
      'from-white bg-linear-to-br to-zinc-200/60 border-zinc-300 shadow-[2px_0_8px_rgba(0,0,0,0.1)]',
      'dark:from-zinc-950 dark:to-zinc-900/60 dark:border-zinc-900/50 dark:shadow-inner',
    )}
  >
    <div class="flex items-center justify-center">
      <svg
        width={SIZE}
        height={SIZE}
        viewBox={`${-SIZE / 2} ${-SIZE / 2} ${SIZE} ${SIZE}`}
      >
        {#each pieData as slice, i (slice.data.label)}
          <path
            d={arcGen(slice) ?? undefined}
            fill={slice.data.label === 'Others' ||
            slice.data.label === 'No Data'
              ? '#3F3F46'
              : COLORS[i]}
            stroke="rgba(255,255,255,0.7)"
            stroke-width="1.2"
          />
        {/each}

        <circle
          r={RADIUS}
          fill="none"
          stroke="rgba(255,255,255,0.7)"
          stroke-width="1.5"
        />
      </svg>
    </div>

    <div>
      {#if title}
        <div class="flex items-center justify-between pr-2">
          <p><span class="font-bold">{title}</span></p>
        </div>
      {/if}

      {#each Object.entries(data)
        .filter(([, value]) => value > 0)
        .filter(([key]) => key !== 'No Data') as [key, value] (key)}
        <p><span class="font-bold">{value}</span> {key}</p>
      {/each}
    </div>
  </div>
</div>
