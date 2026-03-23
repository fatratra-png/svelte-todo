<script lang="ts">
	type Task = {
		id: number;
		title: string;
		desc: string;
		status: 'in-progress' | 'completed' | 'failed';
	};

	let tasks = $state<Task[]>([]);
	let newTitle = $state('');
	let newDesc = $state('');

	const stats = $derived({
		total: tasks.length,
		completed: tasks.filter((t) => t.status === 'completed').length,
		inProgress: tasks.filter((t) => t.status === 'in-progress').length,
		failed: tasks.filter((t) => t.status === 'failed').length
	});

	function addTask() {
		if (!newTitle.trim()) return;
		tasks = [
			...tasks,
			{
				id: Date.now(),
				title: newTitle.trim(),
				desc: newDesc.trim(),
				status: 'in-progress' as keyof typeof statusConfig
			}
		];
		newTitle = '';
		newDesc = '';
	}

	function setStatus(id: number, status: string) {
		tasks = tasks.map((t) =>
			t.id === id ? { ...t, status: status as keyof typeof statusConfig } : t
		);
	}

	function deleteTask(id: number) {
		tasks = tasks.filter((t) => t.id !== id);
	}

	const statusConfig = {
		'in-progress': {
			label: 'In Progress',
			style: 'text-amber-400 border-amber-800 bg-amber-950/40'
		},
		completed: {
			label: 'Completed',
			style: 'text-emerald-400 border-emerald-800 bg-emerald-950/40'
		},
		failed: { label: 'Failed', style: 'text-rose-400 border-rose-800 bg-rose-950/40' }
	};
</script>

<main class="min-h-screen bg-zinc-950 p-8 font-mono text-zinc-100">
	<div class="mx-auto max-w-2xl space-y-8">
		<!-- Header -->
		<div>
			<p class="mb-1 text-xs tracking-[0.3em] text-zinc-600 uppercase">
				Svelte 5 · $state · $derived
			</p>
			<h1 class="text-2xl font-bold tracking-widest">TODO LIST</h1>
		</div>

		<!-- ── SECTION AJOUT ── -->
		<div class="space-y-4 rounded-2xl border border-zinc-800 bg-zinc-900/40 p-6">
			<p class="text-xs tracking-widest text-zinc-500 uppercase">Nouvelle tâche</p>

			<input
				bind:value={newTitle}
				placeholder="Titre de la tâche..."
				class="w-full rounded-xl border border-zinc-800 bg-zinc-950 px-4 py-3 text-sm text-zinc-100
               transition-colors placeholder:text-zinc-700 focus:border-zinc-600 focus:outline-none"
			/>

			<textarea
				bind:value={newDesc}
				placeholder="Description (optionnel)..."
				rows="2"
				class="w-full resize-none rounded-xl border border-zinc-800 bg-zinc-950 px-4 py-3 text-sm
               text-zinc-100 transition-colors placeholder:text-zinc-700 focus:border-zinc-600 focus:outline-none"
			></textarea>

			<button
				onclick={addTask}
				disabled={!newTitle.trim()}
				class="w-full rounded-xl bg-zinc-100 py-3 text-sm font-bold text-zinc-900
               transition-all hover:bg-white active:scale-95
               disabled:cursor-not-allowed disabled:opacity-20 disabled:active:scale-100"
			>
				+ Ajouter la tâche
			</button>
		</div>

		<!-- ── STATS ── -->
		{#if tasks.length > 0}
			<div class="grid grid-cols-4 gap-3">
				{#each [{ label: 'Total', value: stats.total, color: 'text-zinc-300' }, { label: 'In Progress', value: stats.inProgress, color: 'text-amber-400' }, { label: 'Completed', value: stats.completed, color: 'text-emerald-400' }, { label: 'Failed', value: stats.failed, color: 'text-rose-400' }] as stat}
					<div class="rounded-xl border border-zinc-800 bg-zinc-900/50 p-3 text-center">
						<p class="mb-1 text-xs text-zinc-600">{stat.label}</p>
						<p class="text-2xl font-bold {stat.color}">{stat.value}</p>
					</div>
				{/each}
			</div>
		{/if}

		<!-- ── LISTE ── -->
		{#if tasks.length === 0}
			<div class="rounded-2xl border border-dashed border-zinc-800 py-16 text-center">
				<p class="text-sm text-zinc-700">Aucune tâche pour l'instant.</p>
				<p class="mt-1 text-xs text-zinc-800">Ajoute-en une ci-dessus ↑</p>
			</div>
		{:else}
			<div class="overflow-hidden rounded-2xl border border-zinc-800">
				<!-- Header tableau -->
				<div class="grid grid-cols-12 gap-4 border-b border-zinc-800 bg-zinc-900 px-5 py-3">
					<p class="col-span-1 text-xs tracking-widest text-zinc-600 uppercase">#</p>
					<p class="col-span-5 text-xs tracking-widest text-zinc-600 uppercase">Titre</p>
					<p class="col-span-4 text-xs tracking-widest text-zinc-600 uppercase">Status</p>
					<p class="col-span-2 text-right text-xs tracking-widest text-zinc-600 uppercase">Del</p>
				</div>

				<!-- Rows -->
				{#each tasks as task, i (task.id)}
					<div
						class="grid grid-cols-12 items-center gap-4 border-b border-zinc-800/50
                      px-5 py-4 transition-colors
                      last:border-0 hover:bg-zinc-900/40"
					>
						<!-- Index -->
						<span class="col-span-1 text-xs text-zinc-700">{i + 1}</span>

						<!-- Titre + desc -->
						<div class="col-span-5">
							<p
								class="text-sm {task.status === 'completed'
									? 'text-zinc-600 line-through'
									: 'text-zinc-200'}"
							>
								{task.title}
							</p>
							{#if task.desc}
								<p class="mt-0.5 truncate text-xs text-zinc-600">{task.desc}</p>
							{/if}
						</div>

						<!-- Dropdown status -->
						<div class="col-span-4">
							<select
								value={task.status}
								onchange={(e) => setStatus(task.id, e.currentTarget.value)}
								class="w-full cursor-pointer appearance-none rounded-lg border bg-transparent px-2.5
                       py-1.5 text-center text-xs transition-all focus:outline-none
                       {statusConfig[task.status].style}"
							>
								<option value="in-progress" class="bg-zinc-900 text-amber-400"
									>⏳ In Progress</option
								>
								<option value="completed" class="bg-zinc-900 text-emerald-400">✅ Completed</option>
								<option value="failed" class="bg-zinc-900 text-rose-400">❌ Failed</option>
							</select>
						</div>

						<!-- Delete -->
						<div class="col-span-2 flex justify-end">
							<button
								onclick={() => deleteTask(task.id)}
								class="h-7 w-7 rounded-lg text-xs text-zinc-700
                       transition-all hover:bg-rose-950/40 hover:text-rose-400"
							>
								✕
							</button>
						</div>
					</div>
				{/each}
			</div>
		{/if}
	</div>
</main>
