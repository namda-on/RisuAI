<script lang="ts">
	import Suggestion from './Suggestion.svelte';
    import { DatabaseIcon, DicesIcon, LanguagesIcon, MenuIcon, MicOffIcon, PowerIcon, RefreshCcwIcon, ReplyIcon, Send } from "lucide-svelte";
    import { selectedCharID } from "../../ts/stores";
    import Chat from "./Chat.svelte";
    import { DataBase, type Message } from "../../ts/storage/database";
    import { getCharImage } from "../../ts/characters";
    import { doingChat, sendChat } from "../../ts/process/index";
    import { findCharacterbyId, messageForm, sleep } from "../../ts/util";
    import { language } from "../../lang";
    import { translate } from "../../ts/translator/translator";
    import { alertError } from "../../ts/alert";
    import sendSound from '../../etc/send.mp3'
    import {cloneDeep} from 'lodash'
    import { processScript } from "src/ts/process/scripts";
    import CreatorQuote from "./CreatorQuote.svelte";
    import { stopTTS } from "src/ts/process/tts";
    import MainMenu from '../UI/MainMenu.svelte';
    import Help from '../Others/Help.svelte';

    let messageInput:string = ''
    let messageInputTranslate:string = ''
    let openMenu = false
    export let openChatList = false
    let loadPages = 30
    let autoMode = false
    let rerolls:Message[][] = []
    let rerollid = -1
    let lastCharId = -1
    let doingChatInputTranslate = false

    async function send() {
        let selectedChar = $selectedCharID
        console.log('send')
        if($doingChat){
            return
        }
        if(lastCharId !== $selectedCharID){
            rerolls = []
            rerollid = -1
        }

        let cha = $DataBase.characters[selectedChar].chats[$DataBase.characters[selectedChar].chatPage].message

        if(messageInput === ''){
            if($DataBase.characters[selectedChar].type !== 'group'){
                if(cha.length === 0 || cha[cha.length - 1].role !== 'user'){
                    if($DataBase.useSayNothing){
                        cha.push({
                            role: 'user',
                            data: '*says nothing*'
                        })
                    }
                }
            }
        }
        else{
            const char = $DataBase.characters[selectedChar]
            if(char.type === 'character'){
                cha.push({
                    role: 'user',
                    data: processScript(char,messageInput,'editinput')
                })
            }
            else{
                cha.push({
                    role: 'user',
                    data: messageInput
                })
            }
        }
        messageInput = ''
        messageInputTranslate = ''
        $DataBase.characters[selectedChar].chats[$DataBase.characters[selectedChar].chatPage].message = cha
        rerolls = []
        await sleep(10)
        updateInputSizeAll()
        await sendChatMain()

    }

    async function reroll() {
        if($doingChat){
            return
        }
        if(lastCharId !== $selectedCharID){
            rerolls = []
            rerollid = -1
        }
        if(rerollid < rerolls.length - 1){
            if(Array.isArray(rerolls[rerollid + 1])){
                let db = $DataBase
                rerollid += 1
                db.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].message = cloneDeep(rerolls[rerollid])
                $DataBase = db
            }
            return
        }
        if(rerolls.length === 0){
            rerolls.push($DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].message)
            rerollid = rerolls.length - 1
        }
        let cha = $DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].message
        if(cha.length === 0 ){
            return
        }
        openMenu = false
        const saying = cha[cha.length - 1].saying
        let sayingQu = 2
        while(cha[cha.length - 1].role !== 'user'){
            if(cha[cha.length - 1].saying === saying){
                sayingQu -= 1
                if(sayingQu === 0){
                    break
                }   
            }
            cha.pop()
        }
        $DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].message = cha
        await sendChatMain()
    }

    async function unReroll() {
        if(rerollid <= 0){
            return
        }
        if(lastCharId !== $selectedCharID){
            rerolls = []
            rerollid = -1
        }
        if($doingChat){
            return
        }
        if(Array.isArray(rerolls[rerollid - 1])){
            let db = $DataBase
            rerollid -= 1
            db.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].message = cloneDeep(rerolls[rerollid])
            $DataBase = db
        }
    }

    async function sendChatMain(saveReroll = false) {
        messageInput = ''
        try {
            await sendChat()            
        } catch (error) {
            alertError(`${error}`)
        }
        rerolls.push(cloneDeep($DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].message))
        rerollid = rerolls.length - 1
        lastCharId = $selectedCharID
        $doingChat = false
        if($DataBase.playMessage){
            const audio = new Audio(sendSound);
            audio.play();
        }
    }

    async function runAutoMode() {
        if(autoMode){
            autoMode = false
            return
        }
        const selectedChar = $selectedCharID
        autoMode = true
        while(autoMode){
            await sendChatMain()
            if(selectedChar !== $selectedCharID){
                autoMode = false
            }
        }
    }

    export let customStyle = ''
    let inputHeight = "44px"
    let inputEle:HTMLTextAreaElement
    let inputTranslateHeight = "44px"
    let inputTranslateEle:HTMLTextAreaElement

    function updateInputSizeAll() {
        updateInputSize()
        updateInputTranslateSize()
    }

    function updateInputTranslateSize() {
        if(inputTranslateEle) {
            inputTranslateEle.style.height = "0";
            inputTranslateHeight = (inputTranslateEle.scrollHeight) + "px";
            inputTranslateEle.style.height = inputTranslateHeight
        }
    }
    function updateInputSize() {
        if(inputEle){
            inputEle.style.height = "0";
            inputHeight = (inputEle.scrollHeight) + "px";
            inputEle.style.height = inputHeight
        }
    }

    $: updateInputSizeAll()

    function updateInputTransateMessage(reverse: boolean) {
        if(reverse && messageInputTranslate === '') {
            messageInput = ''
            return
        }
        if(!reverse && messageInput === '') {
            messageInputTranslate = ''
            return
        }
        translate(reverse ? messageInputTranslate : messageInput, reverse).then((translatedMessage) => {
            if(translatedMessage){
                if(reverse)
                    messageInput = translatedMessage
                else
                    messageInputTranslate = translatedMessage
            }
        })
    }
</script>
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="w-full h-full" style={customStyle} on:click={() => {
    openMenu = false
}}>
    {#if $selectedCharID < 0}
        <MainMenu />
    {:else}
        <div class="h-full w-full flex flex-col-reverse overflow-y-auto relative"  on:scroll={(e) => {
            //@ts-ignore
            const scrolled = (e.target.scrollHeight - e.target.clientHeight + e.target.scrollTop)
            if(scrolled < 100 && $DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].message.length > loadPages){
                loadPages += 30
            }
        }}>
            <div class="flex items-end mt-2 mb-2 w-full">
                <textarea class="text-neutral-200 p-2 min-w-0 bg-transparent input-text text-xl flex-grow ml-4 mr-2 border-gray-700 resize-none focus:bg-selected overflow-y-hidden overflow-x-hidden max-w-full"
                    bind:value={messageInput}
                    bind:this={inputEle}
                    on:keydown={(e) => {
                        if(e.key.toLocaleLowerCase() === "enter" && (!e.shiftKey)){
                            if($DataBase.sendWithEnter){
                                send()
                                e.preventDefault()
                            }
                        }
                        if(e.key.toLocaleLowerCase() === "m" && (e.ctrlKey)){
                            reroll()
                            e.preventDefault()
                        }
                    }}
                    on:input={()=>{updateInputSizeAll();updateInputTransateMessage(false)}}
                    style:height={inputHeight}
                />

                
                {#if $doingChat || doingChatInputTranslate}
                    <div
                        class="mr-2 bg-selected flex justify-center items-center text-white w-12 h-12 rounded-md hover:bg-green-500 transition-colors">
                        <div class="loadmove" class:autoload={autoMode}>
                        </div>
                    </div>
                {:else}
                    <div on:click={send}
                        class="mr-2 bg-gray-500 flex justify-center items-center text-white w-12 h-12 rounded-md hover:bg-green-500 transition-colors"><Send />
                    </div>
                {/if}
                    <div on:click={(e) => {
                        openMenu = !openMenu
                        e.stopPropagation()
                    }}
                    class="mr-2 bg-gray-500 flex justify-center items-center text-white w-12 h-12 rounded-md hover:bg-green-500 transition-colors"><MenuIcon />
                    </div>
            </div>
            {#if $DataBase.useAutoTranslateInput && $DataBase.useExperimental}
                <div class="flex items-center mt-2 mb-2 w-full">
                    <label for='messageInputTranslate' class="text-neutral-200 ml-4">
                        <LanguagesIcon />
                    </label>
                    <textarea id = 'messageInputTranslate' class="text-neutral-200 p-2 min-w-0 bg-transparent input-text text-xl flex-grow ml-4 mr-2 border-gray-700 resize-none focus:bg-selected overflow-y-hidden overflow-x-hidden max-w-full"
                        bind:value={messageInputTranslate}
                        bind:this={inputTranslateEle}
                        on:keydown={(e) => {
                            if(e.key.toLocaleLowerCase() === "enter" && (!e.shiftKey)){
                                if($DataBase.sendWithEnter){
                                    send()
                                    e.preventDefault()
                                }
                            }
                            if(e.key.toLocaleLowerCase() === "m" && (e.ctrlKey)){
                                reroll()
                                e.preventDefault()
                            }
                        }}
                        on:input={()=>{updateInputSizeAll();updateInputTransateMessage(true)}}
                        placeholder={language.enterMessageForTranslateToEnglish}
                        style:height={inputTranslateHeight}
                    />
                </div>
            {/if}
            
            {#if $DataBase.useAutoSuggestions}
                <Suggestion messageInput={(msg)=>messageInput=msg} {send}/>
            {/if}
            {#each messageForm($DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].message, loadPages) as chat, i}
                {#if chat.role === 'char'}
                    {#if $DataBase.characters[$selectedCharID].type !== 'group'}
                        <Chat
                            idx={chat.index}
                            name={$DataBase.characters[$selectedCharID].name} 
                            message={chat.data}
                            img={getCharImage($DataBase.characters[$selectedCharID].image, 'css')}
                            rerollIcon={i === 0}
                            onReroll={reroll}
                            unReroll={unReroll}
                            isLastMemory={$DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].lastMemory === (chat.chatId ?? 'none') && $DataBase.showMemoryLimit}
                            character={$DataBase.characters[$selectedCharID]}
                        />
                    {:else}
                        <Chat
                            idx={chat.index}
                            name={findCharacterbyId(chat.saying).name} 
                            rerollIcon={i === 0}
                            message={chat.data}
                            onReroll={reroll}
                            unReroll={unReroll}
                            img={getCharImage(findCharacterbyId(chat.saying).image, 'css')}
                            isLastMemory={$DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].lastMemory === (chat.chatId ?? 'none') && $DataBase.showMemoryLimit}
                            character={findCharacterbyId(chat.saying)}
                        />
                    {/if}
                {:else}
                    <Chat
                        character={$DataBase.characters[$selectedCharID]}
                        idx={chat.index}
                        name={$DataBase.username} 
                        message={chat.data}
                        img={getCharImage($DataBase.userIcon, 'css')}
                        isLastMemory={$DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].lastMemory === (chat.chatId ?? 'none') && $DataBase.showMemoryLimit}
                    />
                {/if}
            {/each}
            {#if $DataBase.characters[$selectedCharID].chats[$DataBase.characters[$selectedCharID].chatPage].message.length <= loadPages}
                {#if $DataBase.characters[$selectedCharID].type !== 'group'}
                    <Chat
                        character={$DataBase.characters[$selectedCharID]}
                        name={$DataBase.characters[$selectedCharID].name}
                        message={$DataBase.characters[$selectedCharID].firstMsgIndex === -1 ? $DataBase.characters[$selectedCharID].firstMessage :
                            $DataBase.characters[$selectedCharID].alternateGreetings[$DataBase.characters[$selectedCharID].firstMsgIndex]}
                        img={getCharImage($DataBase.characters[$selectedCharID].image, 'css')}
                        idx={-1}
                        altGreeting={$DataBase.characters[$selectedCharID].alternateGreetings.length > 0}
                        onReroll={() => {
                            const cha = $DataBase.characters[$selectedCharID]
                            if(cha.type !== 'group'){
                                if (cha.firstMsgIndex >= (cha.alternateGreetings.length - 1)){
                                    cha.firstMsgIndex = -1
                                }
                                else{
                                    cha.firstMsgIndex += 1
                                }
                            }
                            $DataBase.characters[$selectedCharID] = cha
                        }}
                        unReroll={() => {
                            const cha = $DataBase.characters[$selectedCharID]
                            if(cha.type !== 'group'){
                                if (cha.firstMsgIndex === -1){
                                    cha.firstMsgIndex = (cha.alternateGreetings.length - 1)
                                }
                                else{
                                    cha.firstMsgIndex -= 1
                                }
                            }
                            $DataBase.characters[$selectedCharID] = cha
                        }}
                        isLastMemory={false}

                    />
                    {#if !$DataBase.characters[$selectedCharID].removedQuotes && $DataBase.characters[$selectedCharID].creatorNotes.length >= 2}
                        <CreatorQuote quote={$DataBase.characters[$selectedCharID].creatorNotes} onRemove={() => {
                            const cha = $DataBase.characters[$selectedCharID]
                            if(cha.type !== 'group'){
                                cha.removedQuotes = true
                            }
                            $DataBase.characters[$selectedCharID] = cha
                        }} />
                    {/if}
                {/if}
            {/if}

            {#if openMenu}
                <div class="absolute right-2 bottom-16 p-5 bg-darkbg flex flex-col gap-3 text-gray-200" on:click={(e) => {
                    e.stopPropagation()
                }}>
                    {#if $DataBase.characters[$selectedCharID].type === 'group'}
                        <div class="flex items-center cursor-pointer hover:text-green-500 transition-colors" on:click={runAutoMode}>
                            <DicesIcon />
                            <span class="ml-2">{language.autoMode}</span>
                        </div>
                    {/if}

                    
                    <!-- svelte-ignore empty-block -->
                    {#if $DataBase.characters[$selectedCharID].ttsMode === 'webspeech' || $DataBase.characters[$selectedCharID].ttsMode === 'elevenlab'}
                        <div class="flex items-center cursor-pointer hover:text-green-500 transition-colors" on:click={() => {
                            stopTTS()
                        }}>
                            <MicOffIcon />
                            <span class="ml-2">{language.ttsStop}</span>
                        </div>
                    {/if}

                    <div class="flex items-center cursor-pointer hover:text-green-500 transition-colors" on:click={() => {
                        openChatList = true
                        openMenu = false
                    }}>
                        <DatabaseIcon />
                        <span class="ml-2">{language.chatList}</span>
                    </div>
                    {#if $DataBase.translator !== ''}
                        <div class="flex items-center cursor-pointer hover:text-green-500 transition-colors" on:click={async () => {
                            doingChatInputTranslate = true
                            messageInput = (await translate(messageInput, true))
                            doingChatInputTranslate = false
                        }}>
                            <LanguagesIcon />
                            <span class="ml-2">{language.translateInput}</span>
                        </div>
                        {#if $DataBase.useExperimental}
                            <div class={"flex items-center cursor-pointer "+ ($DataBase.useAutoTranslateInput ? 'text-green-500':'lg:hover:text-green-500')} on:click={() => {
                                $DataBase.useAutoTranslateInput = !$DataBase.useAutoTranslateInput
                            }}>
                                <LanguagesIcon />
                                <span class="ml-2">{language.autoTranslateInput}</span>
                                <Help key="experimental" />
                            </div>
                        {/if}
                        
                    {/if}
                    
                    <div class={"flex items-center cursor-pointer "+ ($DataBase.useAutoSuggestions ? 'text-green-500':'lg:hover:text-green-500')} on:click={async () => {
                        $DataBase.useAutoSuggestions = !$DataBase.useAutoSuggestions
                    }}>
                        <ReplyIcon />
                        <span class="ml-2">{language.autoSuggest}</span>
                    </div>
                    <div class="flex items-center cursor-pointer hover:text-green-500 transition-colors" on:click={reroll}>
                        <RefreshCcwIcon />
                        <span class="ml-2">{language.reroll}</span>
                    </div>
                </div>

            {/if}
        </div>

    {/if}
</div>
<style>
    .loadmove {
        animation: spin 1s linear infinite;
        border-radius: 50%;
        border: 0.4rem solid rgba(0,0,0,0);
        width: 1rem;
        height: 1rem;
        border-top: 0.4rem solid #6272a4;
        border-left: 0.4rem solid #6272a4;
    }

    .autoload{
        border-top: 0.4rem solid #10b981;
        border-left: 0.4rem solid #10b981;
    }

    @keyframes spin {
        
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
    }
</style>