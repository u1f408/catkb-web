<script lang="ts">
    import HeaderParentLink from '$components/HeaderParentLink.svelte';

    import { storePost, storePatch } from '$lib/storeFetch';
    import { goto, invalidateAll } from '$app/navigation';
    import type { PageData } from './$types';

    export let data: PageData;
    let error: string | null = null;

    async function submit(e: SubmitEvent) {
        e.preventDefault();

        error = null;
        try {
            let fr = Object.fromEntries((new FormData(e.target! as HTMLFormElement)).entries());
            await storePatch({ fetch }, ['package_tracking', data.track_no], fr);
        } catch (ex: any) {
            error = `Failed to update: ${ex.toString()}`;
            return;
        }

        await invalidateAll();
        return await goto(`/packagetrack/${data.track_no}`);
    }

    async function mark(e: SubmitEvent) {
        e.preventDefault();

        error = null;
        try {
            await storePost({ fetch }, ['package_tracking', data.track_no, 'mark'], {});
        } catch (ex: any) {
            error = `Failed to update: ${ex.toString()}`;
            return;
        }

        await invalidateAll();
        return await goto(`/packagetrack/${data.track_no}`);
    }
</script>

<svelte:head>
    <title>Edit package: {data.track_no} ({data.carrier})</title>
</svelte:head>

<h1>
    <HeaderParentLink href="/packagetrack/{data.track_no}" />
    Edit package:
    {#if data.notes}
        {data.notes}
    {/if}{#if !data.notes}
        <code>{data.track_no}</code> ({data.carrier})
    {/if}
</h1>

{#if error}<div class="message error">{error}</div>{/if}

<form on:submit={(e) => submit(e)} class="bigform">
    <label for="notes">Notes:</label>
    <input type="text" id="notes" name="notes" placeholder="Notes" value={data.notes}>

    <label for="completed">
        <input type="checkbox" id="completed" name="completed" checked={data.completed}>
        Completed? (disables automatic status refreshes)
    </label>

    <button type="submit">Update</button>
</form>

<form on:submit={(e) => mark(e)} class="bigform">
    <button type="submit">Mark</button>
</form>
