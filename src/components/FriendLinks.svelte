<script lang="ts">
	interface FriendLink {
		name: string;
		description: string;
		url: string;
		avatar: string;
	}

	let {
		friendLinks = [],
		enabled = true,
	}: {
		friendLinks?: FriendLink[];
		enabled?: boolean;
	} = $props();
</script>

{#if enabled && friendLinks.length > 0}
	<div class="card-base p-4 mb-4">
		<h3 class="text-lg font-bold mb-3 text-neutral-800 dark:text-neutral-200 transition">
			友链
		</h3>
		<div class="space-y-3">
			{#each friendLinks as friend (friend.url)}
				<a
					href={friend.url}
					target="_blank"
					rel="noopener noreferrer"
					class="flex items-center gap-3 p-2 rounded-lg hover:bg-[var(--btn-plain-bg-hover)] active:bg-[var(--btn-plain-bg-active)] transition group active:scale-95"
				>
					<img
						src={friend.avatar}
						alt={friend.name}
						class="w-10 h-10 rounded-full object-cover border-2 border-neutral-300/50
						dark:border-neutral-600/50 transition group-hover:border-[var(--primary)]"
					/>
					<div class="flex-1 min-w-0">
						<div
							class="font-semibold text-sm text-neutral-700 dark:text-neutral-300 transition truncate"
						>
							{friend.name}
						</div>
						<div class="text-xs text-neutral-500 dark:text-neutral-400 transition">
							{friend.description}
						</div>
					</div>
					<div
						class="opacity-0 group-hover:opacity-100 transition text-[var(--primary)]"
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="w-4 h-4"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14"
							/>
						</svg>
					</div>
				</a>
			{/each}
		</div>
	</div>
{/if}
