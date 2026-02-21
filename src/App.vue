<script setup lang="ts">
import { ref, onMounted } from 'vue'

// Interface for shopping list items
interface ShoppingItem {
  id: number
  name: string
  completed: boolean
}

// Reactive state
const items = ref<ShoppingItem[]>([])
const newItemName = ref('')
let nextId = 1

// Callbacks for list state transitions
const onListBecameEmpty = (): void => {
  console.log('List became empty');
  if ('modelContext' in navigator) {
    navigator.modelContext.unregisterTool("remove_item");
  }
}

const onListBecameNonEmpty = (): void => {
  console.log('List became non-empty');
  if ('modelContext' in navigator) {
    navigator.modelContext.registerTool({
      name: "remove_item",
      description: "Removes the given item from the shopping list.",
      inputSchema: {
        type: "object",
        properties: {
          item: {type: "string", description: "The item that should be removed from the list."}
        },
        required: ["item"]
      },
      execute: ({item}) => {
        removeItemByName(item);

        return {
          content: [{
            type: "text", text: JSON.stringify({
              success: true,
              message: "successfully removed item"
            }, null, 2)
          }]
        };
      }
    });
  }
}

// Reusable method to add an item by name
const addItemByName = (name: string): void => {
  if (!name.trim()) return

  const wasEmpty = items.value.length === 0

  items.value.push({
    id: nextId++,
    name: name.trim(),
    completed: false
  })

  if (wasEmpty && items.value.length > 0) {
    onListBecameNonEmpty()
  }
}

// Reusable method to remove an item by name
const removeItemByName = (name: string): void => {
  const index = items.value.findIndex(
    item => item.name.toLowerCase() === name.toLowerCase()
  )

  if (index !== -1) {
    const wasNonEmpty = items.value.length > 0
    items.value.splice(index, 1)

    if (wasNonEmpty && items.value.length === 0) {
      onListBecameEmpty()
    }
  }
}

// Handler for form submission
const handleAddItem = (): void => {
  addItemByName(newItemName.value)
  newItemName.value = ''
}

// Handler for removing item by ID (uses removeItemByName internally)
const handleRemoveItem = (id: number): void => {
  const item = items.value.find(item => item.id === id)
  if (item) {
    removeItemByName(item.name)
  }
}

// Toggle item completion status
const toggleItemCompletion = (id: number): void => {
  const item = items.value.find(item => item.id === id)
  if (item) {
    item.completed = !item.completed
  }
}

// Lifecycle hook - triggered when App is mounted
onMounted(() => {
  console.log('App mounted');
  if ('modelContext' in navigator) {
    navigator.modelContext.registerTool({
      name: "add_item",
      description: "Adds the given item to the shopping list.",
      inputSchema: {
        type: "object",
        properties: {
          item: {type: "string", description: "The item that should be added to the list."}
        },
        required: ["item"]
      },
      execute: ({item}) => {
        addItemByName(item);

        return {
          content: [{
            type: "text", text: JSON.stringify({
              success: true,
              message: "successfully added item"
            }, null, 2)
          }]
        };
      }
    });
  }
})
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 py-12 px-4 sm:px-6 lg:px-8">
    <main class="w-full max-w-md mx-auto">
      <!-- Header -->
      <header class="text-center mb-10">
        <img
          src="/logo.png"
          alt="Shopping List Logo - Purple shopping bag with checklist"
          class="w-24 h-24 mx-auto mb-4 drop-shadow-2xl rounded-3xl"
        />
        <h1 class="text-4xl sm:text-5xl font-bold text-white mb-3 tracking-tight">
          Shopping List
        </h1>
        <p class="text-purple-200 text-lg">
          Keep track of your shopping items
        </p>
      </header>

      <!-- Add Item Form -->
      <section class="bg-white/10 backdrop-blur-lg rounded-2xl shadow-2xl p-6 mb-6 border border-white/20" aria-label="Add new item">
        <form @submit.prevent="handleAddItem" class="flex flex-col sm:flex-row gap-3">
          <label for="new-item-input" class="sr-only">
            Enter item name
          </label>
          <input
            id="new-item-input"
            v-model="newItemName"
            type="text"
            placeholder="Enter item name..."
            class="flex-1 px-4 py-3 bg-white/90 border-0 rounded-xl focus:outline-none focus:ring-2 focus:ring-purple-500 text-gray-800 placeholder-gray-500 shadow-sm"
            aria-label="Item name"
          />
          <button
            type="submit"
            class="px-8 py-3 bg-gradient-to-r from-purple-500 to-pink-500 text-white font-semibold rounded-xl hover:from-purple-600 hover:to-pink-600 focus:outline-none focus:ring-2 focus:ring-purple-500 focus:ring-offset-2 focus:ring-offset-transparent transition-all duration-200 shadow-lg hover:shadow-xl transform hover:-translate-y-0.5"
            aria-label="Add item to list"
          >
            Add Item
          </button>
        </form>
      </section>

      <!-- Shopping List -->
      <section class="bg-white/10 backdrop-blur-lg rounded-2xl shadow-2xl p-6 border border-white/20" aria-label="Shopping list items">
        <div class="flex items-center justify-between mb-6">
          <h2 class="text-2xl font-bold text-white">
            Your Items
          </h2>
          <span class="px-3 py-1 bg-purple-500/30 text-purple-200 rounded-full text-sm font-medium">
            {{ items.length }} {{ items.length === 1 ? 'item' : 'items' }}
          </span>
        </div>

        <!-- Empty state -->
        <div
          v-if="items.length === 0"
          class="text-center py-16"
        >
          <svg
            class="mx-auto h-20 w-20 text-purple-300/50 mb-4"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
            aria-hidden="true"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="1.5"
              d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z"
            />
          </svg>
          <p class="text-purple-100 text-lg font-medium mb-1">
            Your shopping list is empty
          </p>
          <p class="text-purple-300 text-sm">
            Add items above to get started
          </p>
        </div>

        <!-- Items list -->
        <ul
          v-else
          class="space-y-2"
          role="list"
        >
          <li
            v-for="item in items"
            :key="item.id"
            class="group flex items-center gap-3 p-4 bg-white/5 backdrop-blur rounded-xl hover:bg-white/10 transition-all duration-200 border border-white/10"
          >
            <input
              :id="`item-${item.id}`"
              type="checkbox"
              :checked="item.completed"
              @change="toggleItemCompletion(item.id)"
              class="h-5 w-5 text-purple-500 bg-white/20 border-white/30 rounded focus:ring-2 focus:ring-purple-500 cursor-pointer transition-all"
              :aria-label="`Mark ${item.name} as ${item.completed ? 'incomplete' : 'complete'}`"
            />
            <label
              :for="`item-${item.id}`"
              class="flex-1 text-white text-base font-medium cursor-pointer select-none transition-all"
              :class="{ 'line-through text-purple-300 opacity-60': item.completed }"
            >
              {{ item.name }}
            </label>
            <button
              @click="handleRemoveItem(item.id)"
              class="px-3 py-1.5 text-pink-300 hover:text-white hover:bg-pink-500/20 rounded-lg text-sm font-medium transition-all duration-150 focus:outline-none focus:ring-2 focus:ring-pink-500 opacity-0 group-hover:opacity-100"
              :aria-label="`Remove ${item.name} from list`"
            >
              Remove
            </button>
          </li>
        </ul>
      </section>
    </main>
  </div>
</template>
