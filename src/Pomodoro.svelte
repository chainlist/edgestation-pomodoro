<script lang="ts" module>
	type Phase = 'focus' | 'break';

	/** What is saved, so the timer carries on across remounts and restarts. */
	interface Timer {
		phase: Phase;
		/** When the phase ends (epoch ms) while it runs, else `null`. */
		endsAt: number | null;
		/** Time left (ms) while paused, `null` before the phase starts. */
		left: number | null;
	}

	/** Outline icons on a 24×24 box, after Lucide (ISC license). */
	const ICONS = {
		play: 'M7 4.5v15l12.5-7.5z',
		pause: 'M8 5v14M16 5v14',
		reset: 'M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8M3 3v5h5',
		skip: 'M5 4.5v15l10-7.5zM19 5v14'
	};

	const pad = (n: number) => String(n).padStart(2, '0');
</script>

<script lang="ts">
	import type { WidgetProps } from '@edgestation/sdk';

	let props: WidgetProps = $props();
	// `api` never changes: changing a setting remounts the widget.
	// svelte-ignore state_referenced_locally
	const { api } = props;

	const lengths: Record<Phase, number> = {
		focus: Number(api.settings.focus) * 60_000,
		break: Number(api.settings.break) * 60_000
	};

	/** `undefined` until the saved timer has been read. */
	let timer = $state.raw<Timer>();
	let now = $state(Date.now());

	const running = $derived(timer?.endsAt != null);
	const length = $derived(timer ? lengths[timer.phase] : 0);
	// A phase that has started keeps its time left, even if its length setting changed since.
	const left = $derived(
		timer?.endsAt != null ? Math.max(0, timer.endsAt - now) : (timer?.left ?? length)
	);
	const seconds = $derived(Math.ceil(left / 1000));
	const progress = $derived(length ? Math.max(0, 1 - left / length) : 0);

	api.guard(async () => {
		timer = (await api.storage.instance.get<Timer>('timer')) ?? {
			phase: 'focus',
			endsAt: null,
			left: null
		};
	})();

	/** Shows `next` right away and saves it. */
	const save = api.guard(async (next: Timer) => {
		timer = next;
		await api.storage.instance.set('timer', next);
	});

	/** Moves to the other phase, which waits for a tap to start. */
	const next = () =>
		save({ phase: timer!.phase === 'focus' ? 'break' : 'focus', endsAt: null, left: null });

	function toggle() {
		now = Date.now();
		save(running ? { ...timer!, endsAt: null, left } : { ...timer!, endsAt: now + left, left: null });
	}

	const reset = () => save({ ...timer!, endsAt: null, left: null });

	// The time left comes from the clock, not from counting ticks, so it stays right while
	// ticks wait (page off screen, Edge frozen): the first tick after catches up.
	$effect(() => {
		if (!running) return;
		return api.every(250, () => {
			now = Date.now();
			if (left === 0) next();
		});
	});
</script>

{#snippet icon(name: keyof typeof ICONS)}
	<svg viewBox="0 0 24 24" aria-hidden="true"><path d={ICONS[name]} /></svg>
{/snippet}

<!-- `el` is the size container, so its own padding can't use cqh/cqw: lay out inside it. -->
{#if timer}
	<div class="pomodoro">
		<p class="phase">{timer.phase === 'focus' ? 'Focus' : 'Break'}</p>
		<p class="time">{Math.floor(seconds / 60)}:{pad(seconds % 60)}</p>
		<div class="bar"><div class="fill" style:transform="scaleX({progress})"></div></div>
		<div class="controls">
			<button aria-label="Reset" disabled={!running && timer.left === null} onclick={reset}>
				{@render icon('reset')}
			</button>
			<button class="toggle" aria-label={running ? 'Pause' : 'Start'} onclick={toggle}>
				{@render icon(running ? 'pause' : 'play')}
			</button>
			<button
				aria-label={timer.phase === 'focus' ? 'Skip to break' : 'Skip to focus'}
				onclick={next}
			>
				{@render icon('skip')}
			</button>
		</div>
	</div>
{/if}

<style>
	/* cqh/cqw are relative to the widget's cell, so everything scales with its size. */
	.pomodoro {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 3cqh;
		height: 100%;
	}

	p {
		margin: 0;
	}

	.phase {
		font-size: min(9cqh, 5cqw);
		font-weight: 500;
		color: var(--muted-foreground);
	}

	.time {
		font-size: min(32cqh, 16cqw);
		font-weight: 600;
		line-height: 1;
		font-variant-numeric: tabular-nums;
		color: var(--primary);
	}

	.bar {
		width: 60%;
		height: max(3px, 1.5cqh);
		overflow: hidden;
		border-radius: 999px;
		background: var(--muted);
	}

	.fill {
		height: 100%;
		background: var(--primary);
		transform-origin: left;
	}

	.controls {
		display: flex;
		align-items: center;
		gap: min(4cqh, 2.5cqw);
		margin-top: 1cqh;
	}

	button {
		display: grid;
		place-items: center;
		width: min(20cqh, 11cqw);
		aspect-ratio: 1;
		padding: 0;
		border: none;
		border-radius: 50%;
		background: transparent;
		color: var(--card-foreground);
		cursor: pointer;
		transition: transform 0.1s;
	}

	button svg {
		width: 50%;
		fill: none;
		stroke: currentColor;
		stroke-width: 2;
		stroke-linecap: round;
		stroke-linejoin: round;
	}

	.toggle {
		width: min(24cqh, 13cqw);
		background: var(--primary);
		color: var(--primary-foreground);
	}

	button:disabled {
		opacity: 0.35;
		cursor: default;
	}

	button:not(:disabled):active {
		transform: scale(0.9);
	}
</style>
