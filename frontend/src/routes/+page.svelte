<script lang="ts">
	import { goto } from '$app/navigation';
	import { Avatar, ProgressRadial, getToastStore } from '@skeletonlabs/skeleton';
	import { getModalStore } from '@skeletonlabs/skeleton';

	// @ts-expect-error, no types for this module
	import FaArrowRight from 'svelte-icons/fa/FaArrowRight.svelte';
	// @ts-expect-error, no types for this module
	import FaPlus from 'svelte-icons/fa/FaPlus.svelte';
	// @ts-expect-error, no types for this module
	import IoMdExit from 'svelte-icons/io/IoMdExit.svelte';
	// @ts-expect-error, no types for this module
	import FaTimes from 'svelte-icons/fa/FaTimes.svelte';
	// @ts-expect-error, no types for this module
	import FaCheck from 'svelte-icons/fa/FaCheck.svelte';
	import { currentTeam, session, user } from '$lib/storage';
	import { onMount } from 'svelte';
	import { createTeam, inviteReply, type ITeam } from '$lib/api';
	import { fly } from 'svelte/transition';
	import { config } from '$lib/config';

	const modalStore = getModalStore();
	const toastStore = getToastStore();
	let loading = true;

	user.subscribe((value) => {
		if (value.username === '') return;
		loading = false;
	});

	onMount(() => {
		if ($session === null) goto('/login');
	});

	const handleLogOut = () => {
		modalStore.trigger({
			type: 'confirm',
			title: 'Log out',
			body: 'Are you sure you wish to log out from your account?',
			response: (confirmed: boolean) => {
				if (!confirmed) return;
				$session = null;
				goto('/login');
			}
		});
	};

	const handleInviteReply = async (team: ITeam, accepted: boolean) => {
		if ($session === null) return;
		inviteReply($session, team.id, accepted)
			.then(() => {
				$user.invites.splice($user.invites.indexOf(team), 1);
				if (accepted) {
					$user.teams.push(team);
				}
				$user = $user;
			})
			.catch((error) => {
				toastStore.trigger({ message: error, background: 'variant-filled-error' });
			});
	};

	const handleCreateNewTeam = async () => {
		modalStore.trigger({
			type: 'prompt',
			title: 'Create new team',
			body: 'New team name',
			valueAttr: { type: 'text', required: true },
			response: (team_name: string) => {
				if (!team_name || $session === null) return;
				createTeam($session, team_name)
					.then((team) => {
						$user.teams.push(team);
						$user = $user;
					})
					.catch((error) => {
						toastStore.trigger({ message: error, background: 'variant-filled-error' });
					});
			}
		});
	};
</script>

<div class="flex flex-col h-3/4 w-3/4 gap-4">
	<div class="card flex justify-around items-center py-4">
		<div class={loading ? 'animate-pulse' : ''}><Avatar initials={$user.username} /></div>
		<span
			class="text-md text-primary-500 {loading
				? 'placeholder animate-pulse '
				: ''} w-1/3 text-center"
		>
			{$user.username}
		</span>
		<button class="btn btn-icon w-8 text-error-500" on:click={handleLogOut}><IoMdExit /></button>
	</div>

	<div class="flex flex-col m-1 gap-4 overflow-scroll">
		{#each $user.teams as team, index}
			<button
				class="flex justify-between btn card p-4"
				in:fly={{
					duration: config.duration,
					delay: index * config.delay,
					y: config.offset
				}}
				on:click={() => {
					goto(`/team`);
					$currentTeam = team;
				}}
			>
				<span class="font-bold text-xl">{team.name}</span>
				<div class="w-6 text-secondary-500"><FaArrowRight /></div>
			</button>
		{/each}
		{#if loading}
			<div class="w-full flex justify-center">
				<ProgressRadial width="w-12" />
			</div>
		{/if}
		{#each $user.invites as invite, index}
			<div
				class="flex justify-between items-center card p-4"
				in:fly={{
					duration: config.duration,
					delay: (index + $user.teams.length) * config.delay,
					y: config.offset
				}}
			>
				<span class="font-bold text-xl">{invite.name}</span>
				<div class="flex items-center gap-2">
					<button
						class="btn btn-icon w-8 text-error-500"
						on:click={() => handleInviteReply(invite, false)}><FaTimes /></button
					>
					<button
						class="btn btn-icon w-6 text-success-500"
						on:click={() => handleInviteReply(invite, true)}><FaCheck /></button
					>
				</div>
			</div>
		{/each}
		{#if !loading}
			<button
				in:fly={{
					duration: config.duration,
					delay: ($user.invites.length + $user.teams.length) * config.delay,
					y: config.offset
				}}
				class="flex justify-between font-bold btn card p-4"
				on:click={handleCreateNewTeam}
			>
				<span>Create new team</span>
				<div class="w-6 text-success-500"><FaPlus /></div>
			</button>
		{/if}
	</div>
</div>
