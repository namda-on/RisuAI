<!-- svelte-ignore a11y-click-events-have-key-events -->
<div  class="fixed top-0 left-0 h-full w-full bg-black bg-opacity-50 flex flex-col z-50 items-center justify-center" on:click={close}>
    <div class="bg-darkbg rounded-md p-4 max-w-full flex flex-col w-2xl" on:click|stopPropagation>
        <h1 class="font-bold text-2xl w-full">
            <span>
                Share {char.name} to {language.hub}
            </span>
            <button class="float-right text-gray-400 hover:text-green-500" on:click={close}>
                <XIcon />
            </button>
        </h1>
        <div class="mb-2 mt-2 w-full border-t-2 border-t-bgcolor"></div>
        <span class="text-neutral-200">{language.creatorNotes}</span>
        <span class="text-gray-400 text-sm">A description that displays when you search and when you first open a bot.</span>
        <span class="text-gray-400 text-sm">More than 20 characters.</span>
        <textarea class="bg-transparent input-text mt-2 mb-2 text-gray-200 resize-none h-20 min-h-20 focus:bg-selected text-xs w-full" autocomplete="off" bind:value={char.creatorNotes}></textarea>
        <span class="text-neutral-200">{language.tags}</span>
        <span class="text-gray-400 text-sm">Tags to search your character easily. latin alphabets only. seperate by comma.</span>
        <input class="text-neutral-200 mb-4 p-2 bg-transparent input-text focus:bg-selected" placeholder="" bind:value={tags} on:input={() => {
            tags = tags.replace(/[^a-zA-Z,]/g, '').toLocaleLowerCase()
        }}>
        <div class="flex items-center flex-wrap">
            <button class="bg-bgcolor p-2 rounded-lg" class:ring-1={!privateMode} on:click={() => {privateMode = false}}>🌏 Public</button>
            <button class="bg-bgcolor p-2 rounded-lg ml-2" class:ring-1={privateMode} on:click={() => {privateMode = true}}>🔒 Private</button>
        </div>
        <div class="flex items-center flex-wrap mt-2">
            <button class="bg-bgcolor p-2 rounded-lg" class:ring-1={!nsfwMode} on:click={() => {nsfwMode = false}}>🧑‍🧒‍🧒 Safe</button>
            <button class="bg-bgcolor p-2 rounded-lg ml-2" class:ring-1={nsfwMode} on:click={() => {nsfwMode = true}}>🔞 NSFW</button>
        </div>
        {#if nsfwMode}
            <span class="text-gray-400 text-sm">Grotesque Contents and non-adult characters with NSFW would be banned.</span>
        {/if}
        {#if privateMode}
            <span class="text-gray-400 text-sm">Private characters can be removed from the server if there is only a few downloads.</span>
        {/if}
        <button on:click={async () => {
            if(char.creatorNotes.length < 20){
                alertError("Creator Notes must be longer than 20 characters")
            }
            else{
                shareRisuHub(char, {
                    privateMode: privateMode,
                    nsfw: nsfwMode,
                    tag: tags
                })
                close()
            }
        }} class="text-neutral-200 mt-2 text-lg bg-transparent border-solid border-1 border-borderc p-4 hover:bg-green-800 transition-colors cursor-pointer">{language.shareCloud}</button>
    </div>
</div>


<script lang="ts">
    import { XIcon } from "lucide-svelte";
    import { language } from "src/lang";
  import { alertError } from "src/ts/alert";
  import { shareRisuHub } from "src/ts/characterCards";
    import type { character } from "src/ts/storage/database";
    export let close = () => {}
    export let char:character
    let tags=""
    let privateMode = false
    let nsfwMode = false

</script>