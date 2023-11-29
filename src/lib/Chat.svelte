<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { afterUpdate, beforeUpdate, onMount } from 'svelte';
    import { ProgressRadial } from '@skeletonlabs/skeleton';
    import snarkdown from 'snarkdown';
    import { afterNavigate } from '$app/navigation';
    import type { Chat } from '$misc/shared';
    import { chatStore, enhancedLiveAnswerStore, isLoadingAnswerStore } from '$misc/stores';
    import ChatMessages from './ChatMessages.svelte';

    export let slug: string;
    export let chat: Chat | undefined = undefined;

    $: if ($chatStore[slug]) {
        chat = $chatStore[slug];
    }

    let div: HTMLElement | null | undefined;
    let autoscroll: boolean | null | undefined;
    let timer: ReturnType<typeof setTimeout>;
    let isTimeUp = false;

    onMount(() => {
        div = document.getElementById('page');

        timer = setTimeout(() => {
            isTimeUp = true;
            console.log("Time is up!");
        }, 1 * 60 * 1000); // 30 minutes in milliseconds
    });

    onDestroy(() => {
        clearTimeout(timer);
    });

    beforeUpdate(() => {
        autoscroll = div && div.offsetHeight + div.scrollTop > div.scrollHeight - 20;
    });

    afterUpdate(() => {
        if (autoscroll) div?.scrollTo({ top: div.scrollHeight, behavior: 'smooth' });
    });

    afterNavigate(() => {
        div?.scrollTo({ top: div.scrollHeight, behavior: 'smooth' });
    });
</script>

{#if chat}
    <div class="flex flex-col container h-full mx-auto px-4 md:px-8" style="justify-content: end">
        <slot name="additional-content-top" />

        <div class="flex flex-col max-w-4xl md:mx-auto space-y-6 pt-6">
            <ChatMessages {slug} siblings={chat.messages} on:editMessage />

            {#if $isLoadingAnswerStore}
                <div class="place-self-start">
                    <div class="p-5 rounded-2xl variant-ghost-tertiary rounded-tl-none">
                        {@html snarkdown($enhancedLiveAnswerStore.content)}
                    </div>
                </div>
            {/if}
        </div>

        <slot name="additional-content-bottom" />

        <div class="animate-pulse md:w-12 self-center py-2 md:py-6" class:invisible={!$isLoadingAnswerStore}>
            <ProgressRadial class="w-8" stroke={120} meter="stroke-tertiary-500" track="stroke-tertiary-500/30" />
        </div>

        {#if !isTimeUp}
            <!-- Chat input component here -->
        {:else}
            <div class="chat-disabled-message">
                The chat session has ended. Thank you for participating.
            </div>
        {/if}
    </div>
{/if}
