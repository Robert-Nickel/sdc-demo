<script lang="ts">
    import { onMount } from "svelte";
    import { db } from "$lib/supabaseClient";
    import type { User } from "@supabase/supabase-js";

    let items: any[] = [];
    let error: string | null = null;
    let user: User | null | undefined = null;

    onMount(async () => {
        const {
            data: { session },
        } = await db.auth.getSession();
        user = session?.user;
        const { data, error: err } = await db.from("evaluations").select("*");

        if (err) {
            error = err.message;
            console.error("Supabase Error:", err);
        } else {
            items = data;
            console.log("Fetched Items:", data);
        }
    });
</script>

{#if error}
    <p>Error: {error}</p>
{:else if items.length === 0}
    <p>Loading...</p>
{:else}
    <ul>
        {#each items as item}
            <li>{item.name} rates with {item.rating}</li>
        {/each}
    </ul>
{/if}

{#if user}
    <a href="/rate">Ich will auch ein Bewertung abgeben.</a>

    <br><br><br><button
    on:click={() => {
        db.auth.signOut();
    }}>Ausloggen</button
>

{:else}
    <a href="/login">Einloggen, um eine Bewertung abgeben.</a>
{/if}

