<script lang="ts">
    import type { StreamingMessage } from '$lib/types';
    export let message: StreamingMessage;

    // Helper to format timestamp if we add it later
    const formatTime = (date: Date) => {
        return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    };
</script>

<div class="mb-4 last:mb-0 {message.role === 'assistant' ? 'pl-2' : 'pl-2'}">
    <div class="flex items-start gap-3">
        <div 
            class="w-8 h-8 rounded-full flex items-center justify-center shrink-0 text-sm font-medium
            {message.role === 'assistant' 
                ? 'bg-purple-100 text-purple-600' 
                : 'bg-blue-100 text-blue-600'}"
        >
        </div>
        <div class="flex-1 max-w-[85%]">
            <div class="flex items-center gap-2 mb-1">
                <span class="font-medium text-white">
                    {message.role === 'assistant' ? 'Sellen' : 'You'}
                </span>
                <span class="text-xs text-white">
                    {formatTime(new Date())}
                </span> 
               
            </div>
            <div class="rounded-lg px-4 py-2.5 shadow-sm
                {message.role === 'assistant' 
                    ? 'bg-white border border-purple-200' 
                    : 'bg-blue-500 text-white'}"
            >
                <p class="whitespace-pre-wrap break-words text-[15px] leading-relaxed">
                    {message.content}
                    {#if message.isStreaming}
                        <!-- svelte-ignore element_invalid_self_closing_tag -->
                        <span class="inline-block w-1.5 h-4 ml-0.5 align-middle animate-pulse bg-current opacity-75" />
                    {/if}
                </p>
            </div>
        </div>
    </div>
</div> 