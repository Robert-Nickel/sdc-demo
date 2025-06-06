<script lang="ts">
    import { onMount, onDestroy } from "svelte";
    import { db } from "$lib/supabaseClient";
    import type { User } from "@supabase/supabase-js";
    import type { RealtimeChannel } from "@supabase/supabase-js";

    let evaluations: any[] = [];
    let channel: RealtimeChannel;

    let user: User | null | undefined = null;
    let email = "";

    let message = "";
    let error: string | null = null;

    let name = "";
    let rating: number = 1;
    let comment = "";

    let rated = false;

    onMount(async () => {
        const {
            data: { session },
        } = await db.auth.getSession();
        user = session?.user;
        await loadEvaluations();

        channel = db
            .channel("public:evaluations")
            .on(
                "postgres_changes",
                { event: "INSERT", schema: "public", table: "evaluations" },
                (payload) => {
                    const newEval = payload.new;
                    evaluations = [...evaluations, newEval];
                },
            )
            .subscribe();
    });

    onDestroy(() => {
        if (channel) db.removeChannel(channel);
    });

    async function loadEvaluations() {
        const { data, error: err } = await db.from("evaluations").select("*");
        if (err) {
            error = err.message;
            console.error("Supabase Error:", err);
        } else {
            evaluations = data;
        }
    }

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
            comment,
        });

        message = error ? "Error: " + error.message : "Bewertung eingereicht!";
        rated = true;
        await loadEvaluations();
    }

    function getAverageRating(): string {
        if (evaluations.length === 0) return "–";
        const sum = evaluations.reduce((acc, curr) => acc + curr.rating, 0);
        const avg = sum / evaluations.length;
        return avg.toFixed(1);
    }
</script>

<h1>Bewertungen</h1>
<p>Ø Bewertung: {getAverageRating()} / 5</p>

{#if error}
    <p>Error: {error}</p>
{:else if evaluations.length === 0}
    <p>Noch keine Bewertungen</p>
{:else}
    <ul>
        {#each evaluations as e}
            <li>{e.name} rated {e.rating}</li>
        {/each}
    </ul>
{/if}
{#if user}
    {#if rated === false}
        <form on:submit|preventDefault={addItem}>
            <label
                >Wie heißt du?<input
                    bind:value={name}
                    placeholder="Jon Doe"
                    required
                /></label
            >
            <label>
                Auf einer Skala von 1 bis 5, wie fandest du den Talk?
                <input
                    bind:value={rating}
                    type="number"
                    min="1"
                    max="5"
                    step="1"
                    placeholder="Bewertung"
                    required
                />
            </label>
            <label>
                (Optionaler) Kommentar <input
                    bind:value={comment}
                    placeholder="Mir hat gefehlt, ..."
                /></label
            >
            <button type="submit">Bewerten</button>
        </form>
    {:else}
        <h3>Danke für deine Bewertung.</h3>
    {/if}
    <br /><br /><br />

    <button
        class="outline secondary"
        on:click={async () => {
            await db.auth.signOut();
            user = null;
            rated = false;
        }}
    >
        Ausloggen
    </button>
{:else}
    <label>
        E-Mail-Adresse zum mitmachen
        <input
            type="email"
            bind:value={email}
            placeholder="jon@doe.org"
            required
        />
    </label>
    <button on:click={sendMagicLink}>Schick mir Magie!</button>
    <p>{message}</p>
{/if}
