<script>
	import { onMount } from 'svelte';
	import { session } from '$store/session';
	import { goto } from '$app/navigation';
	import { signOut } from 'firebase/auth';
	import { auth } from '$lib/firebase.client';

	// import type { LayoutData } from './$types';
	export let data;

	let loading;
	let loggedIn;

	session.subscribe((cur) => {
		loading = cur?.loading;
		loggedIn = cur?.loggedIn;
	});

	onMount(async () => {
		const user = await data.getAuthUser();

		const loggedIn = !!user && user?.emailVerified;
		session.update((cur) => {
			loading = false;
			return {
				...cur,
				user,
				loggedIn,
				loading: false
			};
		});

		if (loggedIn) {
			goto('/');
		}
	});

	export function logout() {
		signOut(auth)
			.then(() => {
				goto('/login');
				loggedIn = false;
			})
			.catch((error) => {
				throw new Error(error);
			});
	}
</script>

{#if loading}
	<div>Loading...</div>
{:else}
	<div>
		Logged in: {loggedIn}

		<div>
			{#if loggedIn}
				<button on:click={logout}>Logout</button>
			{:else}
				<a href="/login"> Login</a>
			{/if}
		</div>
		<slot />
	</div>
{/if}
