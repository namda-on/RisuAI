<script lang="ts">
    import { DataBase } from "src/ts/storage/database";
    import { getHordeModels } from "src/ts/horde/getModels";
    import Arcodion from "./Arcodion.svelte";
    import { language } from "src/lang";
  import { isNodeServer, isTauri } from "src/ts/storage/globalApi";

    export let value = ""
    export let onChange: (v:string) => void = (v) => {}
    let openOptions = false

    function getModelName(name:string){
        switch(name){
            case "gpt35":
                return "GPT-3.5 Turbo"
            case "gpt4":
                return "GPT-4"
            case "gpt4_32k":
                return "GPT-4 32k"
            case "palm2":
                return "PaLM2"
            case "textgen_webui":
                return "Oobabooga WebUI"
            case "kobold":
                return "Kobold"
            case "custom":
                return "Plugin"
            case "novelai":
                return "NovelAI"
            default:
                if(name.startsWith("horde:::")){
                    return name.replace(":::", " ")
                }
                return name
        }
    }

    function changeModel(name:string){
        value = name
        openOptions = false
        onChange(name)
    }
</script>

{#if openOptions}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="fixed top-0 w-full h-full left-0 bg-black bg-opacity-50 z-50 flex justify-center items-center" on:click={() => {
        openOptions = false
    }}>
        <div class="w-96 max-w-full max-h-full overflow-y-auto overflow-x-hidden bg-bgcolor p-4 flex flex-col" on:click|stopPropagation>
            <h1 class="font-bold text-xl">{language.model}</h1>
            <div class="border-t-1 border-y-selected mt-1 mb-1"></div>
            <Arcodion name="OpenAI GPT">
                <button class="p-2 hover:text-green-500" on:click={() => {changeModel('gpt35')}}>GPT-3.5 Turbo</button>
                <button class="p-2 hover:text-green-500" on:click={() => {changeModel('gpt4')}}>GPT-4</button>
                <button class="p-2 hover:text-green-500" on:click={() => {changeModel('gpt4_32k')}}>GPT-4 32K</button>
            </Arcodion>
            <Arcodion name="Anthropic Claude">
                <button class="p-2 hover:text-green-500" on:click={() => {changeModel('claude-v1')}}>claude-v1</button>
                <button class="p-2 hover:text-green-500" on:click={() => {changeModel('claude-v1-100k')}}>claude-v1-100k</button>
                <button class="p-2 hover:text-green-500" on:click={() => {changeModel('claude-instant-v1')}}>claude-instant-v1</button>
                <button class="p-2 hover:text-green-500" on:click={() => {changeModel('claude-instant-v1-100k')}}>claude-instant-v1-100k</button>
            </Arcodion>
            <button class="hover:bg-selected px-6 py-2 text-lg" on:click={() => {changeModel('textgen_webui')}}>Oobabooga WebUI</button>
            <button class="hover:bg-selected px-6 py-2 text-lg" on:click={() => {changeModel('palm2')}}>Google PaLM2</button>
            <button class="hover:bg-selected px-6 py-2 text-lg" on:click={() => {changeModel('kobold')}}>Kobold</button>
            {#if isTauri ||isNodeServer}
                <button class="hover:bg-selected px-6 py-2 text-lg" on:click={() => {changeModel('novelai')}}>NovelAI Clio</button>
            {/if}
            <Arcodion name="Horde">
                {#await getHordeModels()}
                    <button class="p-2">Loading...</button>
                {:then models}
                    {#each models as model}
                        <button on:click={() => {changeModel("horde:::" + model)}} class="p-2 hover:text-green-500">{model.trim()}</button>
                    {/each}
                {/await}
            </Arcodion>
            {#if $DataBase.plugins.length > 0}
                <button on:click={() => {changeModel('custom')}} class="hover:bg-selected px-6 py-2 text-lg" >Plugin</button>
            {/if}
        </div>
    </div>

{/if}

<button on:click={() => {openOptions = true}}
    class="mt-4 drop-shadow-lg p-3 flex justify-center items-center ml-2 mr-2 rounded-lg bg-selected mb-4">
        {getModelName(value)}
</button>

