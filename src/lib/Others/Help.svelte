

<button class="relative help inline-block cursor-default hover:text-green-500" on:click={() => {
    alertMd(language.help[key])
}}>
    {#if key === "experimental"}
        <div class="text-red-500 hover:text-green-500">
            <FlaskConicalIcon size={16} />
        </div>
    {:else if unrecommended}
        <div class="text-red-500 hover:text-green-500">
            <AlertTriangle size={14} />
        </div>
    {:else}
        <HelpCircleIcon size={14} />
    {/if}
</button>

<script lang="ts">
    import { AlertTriangle, FlaskConicalIcon, HelpCircleIcon } from "lucide-svelte";
    import { language } from "src/lang";
    import { alertMd } from "src/ts/alert";

    export let unrecommended = false
    export let key: (keyof (typeof language.help))
</script>