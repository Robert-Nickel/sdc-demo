<script lang="ts">
    import { goto } from "$app/navigation";
    import { db } from "$lib/supabaseClient";
    import type { User } from "@supabase/supabase-js";
    import { onMount } from "svelte";

    let name = "";
    let rating: number;
    let user: User | null | undefined = null;
    let message = "";

    onMount(async () => {
        const {
            data: { session },
        } = await db.auth.getSession();
        user = session?.user;
    });

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

        goto("/");
    }
</script>

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
<p>{message}</p>
