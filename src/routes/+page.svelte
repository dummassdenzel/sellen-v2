<script lang="ts">
    import { onMount } from 'svelte';
    import ChatMessage from '$lib/components/ChatMessage.svelte';
    import type { StreamingMessage } from '$lib/types';
    import { findRelevantEntries, formatContext } from '$lib/data/dataset';

    let messages: StreamingMessage[] = [];
    let userInput = '';
    let isLoading = false;
    let selectedModel = 'mistral';

    function addMessage(role: 'user' | 'assistant', content: string, isStreaming = false) {
        messages = [...messages, { role, content, isStreaming }];
    }

    async function handleSubmit() {
        if (!userInput.trim()) return;
        
        const userMessage = userInput.trim();
        addMessage('user', userMessage);
        userInput = '';
        isLoading = true;

        // Find relevant context from dataset
        const relevantEntries = findRelevantEntries(userMessage);
        const context = formatContext(relevantEntries);
        
        try {
            const response = await fetch('/api/chat', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ 
                    message: userMessage, 
                    context,
                    model: selectedModel
                })
            });

            if (!response.ok) throw new Error('Failed to get response');
            
            
            addMessage('assistant', '', true);
            
            const reader = response.body?.getReader();
            if (!reader) throw new Error('No reader available');
            
            let accumulatedResponse = '';
            
            while (true) {
                const { done, value } = await reader.read();
                if (done) break;
                
                const chunk = new TextDecoder().decode(value);
                accumulatedResponse += chunk;
                
                messages = messages.map((msg, i) => 
                    i === messages.length - 1 
                        ? { ...msg, content: accumulatedResponse } 
                        : msg
                );
            }
            
            messages = messages.map((msg, i) => 
                i === messages.length - 1 
                    ? { ...msg, isStreaming: false } 
                    : msg
            );
        } catch (error) {
            console.error('Error:', error);
            addMessage('assistant', 'Sorry, I encountered an error while processing your request.');
        } finally {
            isLoading = false;
        }
    }

</script>

<div class="min-h-screen bg-[#222831]  flex flex-col">
   

    <main class="flex-1 w-full mx-auto">
        <div class="bg-[#222831] shadow-lg h-[100vh] flex flex-col">
            <div class="flex-1 overflow-y-auto p-4">
                {#if messages.length === 0}
                    <div class="h-full flex items-center justify-center text-white">
                        <h1 class="text-4xl font-medium">Hello! How can I be of assistance today?</h1>
                    </div>
                {:else}
                    {#each messages as message}
                        <ChatMessage {message} />
                    {/each}
                {/if}
            </div>
            
            <div class="p-4">
                <form on:submit|preventDefault={handleSubmit} class="flex gap-2">
                    <input
                        type="text"
                        bind:value={userInput}
                        placeholder="Ask me about anything about you!"
                        disabled={isLoading}
                        class="flex-1 border text-black bg-white rounded-lg px-4 py-2"
                    />
                    <button
                        type="submit"
                        disabled={isLoading || !userInput.trim()}
                        class="bg-blue-900 text-white rounded-lg px-4 py-2 font-medium hover:bg-blue-700  disabled:opacity-50 disabled:cursor-not-allowed"
                    >
                        {isLoading ? 'Sending...' : 'Ask'}
                    </button>
                </form>
            </div>
        </div>
    </main>
</div>
