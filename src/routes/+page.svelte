<script lang="ts">
    import { onMount } from "svelte";
    import { db } from "$lib/supabaseClient";
    import type { User } from "@supabase/supabase-js";

    let items: any[] = [];
    let error: string | null = null;
    let user: User | null | undefined = null;
    let email = "";
    let message = "";
    let name = "";
    let rating: number = 0;

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

    async function sendMagicLink() {
        const { error } = await db.auth.signInWithOtp({ email });

        if (error) {
            message = "Failed to send link: " + error.message;
        } else {
            message = "Magic link gesendet! Check deine E-Mails.";
        }
    }

    async function addItem() {
        if (!user) {
            message = "Bitte einloggen, um Roberts Talk zu bewerten.";
            return;
        }

        const { error } = await db.from("evaluations").insert({
            name,
            rating,
        });

        message = error ? "Error: " + error.message : "Bewertung eingereicht!";
    }
</script>

{#if error}
    <p>Error: {error}</p>
{:else if items.length === 0}
    <p>Loading...</p>
{:else}
    <h1>Alle Bewertungen für Roberts Talk</h1>
    <ul>
        {#each items as item}
            <li>{item.name} rated {item.rating}</li>
        {/each}
    </ul>
{/if}

{#if user}
    <form on:submit|preventDefault={addItem}>
        <input bind:value={name} placeholder="Dein Name" required />
        <input
            bind:value={rating}
            type="number"
            min="1"
            max="5"
            step="1"
            placeholder="Bewertung"
            required
        />
        <button type="submit">Bewerten</button>
    </form>

    <br /><br /><br /><button class="outline secondary"
        on:click={() => {
            db.auth.signOut();
        }}>Ausloggen</button
    >
{:else}
    <!-- <a href="/login">Einloggen, um eine Bewertung abgeben.</a>-->
    <input
        type="email"
        bind:value={email}
        placeholder="E-Mail-Adresse zum mitmachen"
    />
    <button on:click={sendMagicLink}>Send Magic Link</button>
    <p>{message}</p>
{/if}
