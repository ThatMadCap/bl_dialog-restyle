<script lang="ts">
    import { onMount } from 'svelte';
    import { Receive } from '@enums/events';
    import { ReceiveEvent, SendEvent } from '@utils/eventsHandlers';
    import HexSVG from '@components/hex.svg';
    import CircleSVG from '@components/circle-solid.svg';

    interface Buttons {
        label: string;
        id?: string | number;
    }
    interface Dialogue {
        job: string;
        name: string;
        text: string;
        buttons: Buttons[];
    }

    let currentDialogue: Dialogue = {
        job: '',
        name: '',
        text: '',
        buttons: [],
    };

    let displayedText = '';
    let index = 0;
    let finish = false;
    let lineWidth = '0px';

    function typeWriterEffect() {
        if (index < currentDialogue.text.length) {
            displayedText += currentDialogue.text.charAt(index);
            index++;
            setTimeout(typeWriterEffect, 25);
        } else finish = true;
    }

    function refreshText() {
        index = 0
        displayedText = ''
        typeWriterEffect()
    }

    function selectButton(index: number, id?: string | number) {
        finish = false;
        SendEvent('dialog:click', {
            index,
            id,
        })
    }

    function calculateLineWidth() {
        const span = document.createElement('span');
        span.style.font = '700 42px Arial';
        span.textContent = currentDialogue.name;
        document.body.appendChild(span);
        lineWidth = `${span.offsetWidth + 25}px`;
        document.body.removeChild(span);
    }

    ReceiveEvent(Receive.showDialogue, (data: Dialogue) => {
        currentDialogue = data;
        refreshText()
        calculateLineWidth();
    });

    ReceiveEvent(Receive.updateDialogue,(data: { type: string; value: string | Buttons[] }) => {
        currentDialogue = { ...currentDialogue, [data.type]: data.value }
        if (data.type === 'text') refreshText()
    });

</script>

<div class="w-full h-[60%] absolute bg-gradient-to-t from-yume_black to-transparent"></div>
<div class="w-[35%] h-[40%] z-10">

    <p class="text-yume_blue font-[600] leading-none">{currentDialogue.job}</p>

    <p class="text-yume_white text-[42px] font-[700] mb-2.5 leading-none">{currentDialogue.name}</p>

    <div class="h-[5px] bg-gradient-to-r from-yume_blue to-yume_pink mb-10" style="width: {lineWidth};"></div>

    <div class="dialog-background">
        <p class="break-words overflow-wrap break-word">{@html displayedText}</p>
    </div>

    <div class="grid grid-cols-2 gap-4 w-[90%] mt-5">
        {#if finish}
            {#each currentDialogue.buttons as item, index}
                <button
                    on:click={() => selectButton(index + 1, item.id)}
                    class="dialog-button flex items-center gap-5 break-words overflow-wrap break-word"
                >
                    <img src={CircleSVG} alt="Circle Icon" width="10" height="10" />
                    {item.label}
                </button>
            {/each}
        {/if}
    </div>
</div>
