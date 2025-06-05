<script lang="ts">
    import { db } from "$lib/supabaseClient";

    let email = "";
    let message = "";

    async function sendMagicLink() {
        const { error } = await db.auth.signInWithOtp({ email });

        if (error) {
            message = "Failed to send link: " + error.message;
        } else {
            message = "Magic link gesendet! Check deine E-Mails.";
        }
    }
</script>

<input type="email" bind:value={email} placeholder="Deine E-Mailadresse" />
<button on:click={sendMagicLink}>Send Magic Link</button>
<p>{message}</p>
