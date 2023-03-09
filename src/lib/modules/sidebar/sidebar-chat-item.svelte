<script lang="ts">
  import { getContext, tick } from 'svelte';
  import { page as page$ } from '$app/stores';
  import { goto } from '$app/navigation';

  import ChatBubbleLeftIcon from '$lib/shared/icons/chat-bubble-left-icon.svelte';
  import { truncateString } from '$lib/shared/shared-utils';
  import { chatList$, chats$ } from '$lib/shared/shared.store';
  import { LOCAL_STORAGE_KEY } from '$lib/shared/shared.type';
  import PencilSquareIcon from '$lib/shared/icons/pencil-square-icon.svelte';
  import TrashIcon from '$lib/shared/icons/trash-icon.svelte';
  import CheckIcon from '$lib/shared/icons/check-icon.svelte';
  import XMarkIcon from '$lib/shared/icons/x-mark-icon.svelte';

  let { handleCloseMobileSidebar } = getContext('sidebar') as any;

  export let chatId;
  export let title;

  let isEditing = false;
  let titleInput = title;

  const handleTitleEditClick = () => {
    isEditing = true;
  };

  const handleCancelTitleEditClick = () => {
    isEditing = false;
  };

  const handleSaveTitleEditClick = () => {
    chatList$.update((chatList) => {
      chatList = chatList.map((chat) => {
        if (chat.chatId === chatId) {
          chat.title = titleInput;
        }
        return chat;
      });
      return chatList;
    });
    localStorage.setItem(LOCAL_STORAGE_KEY.CHAT_LIST, JSON.stringify($chatList$));
    isEditing = false;
  };

  /**
   * Delete chat
   */
  const handleDeleteChat = (chatId: string) => {
    chatList$.update((chatList) => {
      chatList = chatList.filter((chat) => chat.chatId !== chatId);
      return chatList;
    });
    chats$.update((chats) => {
      delete chats[chatId];
      return chats;
    });

    localStorage.setItem(LOCAL_STORAGE_KEY.CHAT_LIST, JSON.stringify($chatList$));
    localStorage.removeItem(chatId);

    if ($page$.params?.chatId === chatId) {
      goto('/');
    }
  };
</script>

<button
  on:click={(e) => {
    if (isEditing) {
      return;
    }

    goto(`/chat/${chatId}`);
    handleCloseMobileSidebar();
  }}
  type="button"
  class={`w-full text-gray-600 hover:bg-gray-100 hover:text-gray-900 group flex items-center px-2 py-3 text-sm font-medium rounded-md ${
    chatId === $page$?.params?.chatId ? `text-gray-900 bg-gray-200` : ''
  }`}
>
  <!-- Title  -->
  <div class="flex flex-1 justify-start items-center flex-nowrap">
    <ChatBubbleLeftIcon
      overrideClasses={`text-gray-400 hover:text-gray-500 mr-3 flex-shrink-0 h-5 w-5`}
    />
    {#if isEditing}
      <input
        bind:value={titleInput}
        on:click={(e) => e.stopPropagation()}
        type="text"
        name="title"
        class="block w-full mr-3 rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-indigo-600 sm:text-sm sm:leading-6"
      />
    {:else}
      <span
        class="text-left overflow-hidden whitespace-nowrap"
        {title}
      >
        {truncateString(title)}
      </span>
    {/if}
  </div>

  <!-- Actions -->
  {#if isEditing}
    <div class="flex gap-2">
      <button on:click={() => handleSaveTitleEditClick()}>
        <CheckIcon overrideClasses={`text-gray-400 hover:text-gray-900 h-3.5 w-3.5`} />
      </button>
      <button on:click={() => handleCancelTitleEditClick()}>
        <XMarkIcon overrideClasses={`text-gray-400 hover:text-gray-900 h-3.5 w-3.5`} />
      </button>
    </div>
  {:else}
    <div class="flex gap-2">
      <button on:click={() => handleTitleEditClick()}>
        <PencilSquareIcon
          overrideClasses={`text-gray-400 hover:text-gray-900 h-3.5 w-3.5`}
        />
      </button>
      <button on:click={() => handleDeleteChat(chatId)}>
        <TrashIcon overrideClasses={`text-gray-400 hover:text-gray-900 h-3.5 w-3.5`} />
      </button>
    </div>
  {/if}
</button>