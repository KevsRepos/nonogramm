<script lang="ts">
import { IconArrowLeft } from "@tabler/icons-svelte";
import "../app.pcss";
import {AppShell, AppBar} from "@skeletonlabs/skeleton";
import { page } from "$app/stores";

// @ts-ignore
const {children} = $props();
</script>

<AppShell>
    <AppBar slot="header">
        <svelte:fragment slot="lead">
            {#if $page.route.id !== '/'}
                <a href="/"><IconArrowLeft /></a>
            {/if}
        </svelte:fragment>

        Nonogramm
    </AppBar>

    {@render children()}
</AppShell>