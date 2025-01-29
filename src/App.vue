<script>
import logService from "./service/logService";
import { ref, computed } from 'vue';

export default {
  name: 'KeywordGenerator',
  setup() {

    //logService.setup();
    //logService.sendLog("search", "keyword");

    const keyword = ref('');
    const relatedPhrases = ref([]);
    const loading = ref(false);
    const error = ref('');
    const sortKey = ref('keyword'); // Default sort key
    const sortOrder = ref('asc'); // Default sort order

    const selectedSpeed = ref('fast'); // Set default speed to "fast"

    // Define token limits based on speed
    const tokenLimits = {
      fast: 50,
      medium: 100,
      slow: 200,
    };

    const fetchKeywords = async () => {
      loading.value = true;
      error.value = '';

      try {
        const tokens = tokenLimits[selectedSpeed.value]; // Get token limit based on selected speed
  
        const response = await fetch(`https://keywordio-d7419b16e33c.herokuapp.com/api/related-searches?keyword=${keyword.value}&speed=${tokens}`);
        if (!response.ok) {
          throw new Error('Failed to fetch keywords');
        }

        const data = await response.json();
        console.log('Response data:', data);
        relatedPhrases.value = data.results;

      } catch (err) {
        error.value = err.message;
      } finally {
        loading.value = false;
      }

    };

    const getIntentClass = (intent) => {
      switch (intent) {
        case 'commercial':
          return 'bg-green-500';
        case 'transactional':
          return 'bg-blue-500';
        case 'navigational':
          return 'bg-purple-500';
        case 'informational':
          return 'bg-yellow-500';
        case 'local':
          return 'bg-red-500';
        case 'investigational':
          return 'bg-orange-500';
        default:
          return 'bg-gray-500';
      }
    };


    const performSearch = (clickedKeyword) => {
      keyword.value = clickedKeyword; // Set the input field to the clicked keyword
      fetchKeywords(clickedKeyword); // Trigger a new search with the clicked keyword
    };

    const sortTable = (key) => {
      sortOrder.value = (sortKey.value === key && sortOrder.value === 'asc') ? 'desc' : 'asc';
      sortKey.value = key;
    };

    const sortedPhrases = computed(() => {
      return [...relatedPhrases.value].sort((a, b) => {
        const modifier = sortOrder.value === 'asc' ? 1 : -1;
        if (sortKey.value === 'volume') {
          return (Number(a.volume.data[0].vol) - Number(b.volume.data[0].vol)) * modifier; // Compare as numbers
        }
        if (a[sortKey.value] < b[sortKey.value]) return -1 * modifier;
        if (a[sortKey.value] > b[sortKey.value]) return 1 * modifier;
        return 0;
      });
    });

    const copyToClipboard = () => {
      const keywords = sortedPhrases.value.map(phrase => phrase.keyword).join('\n');
      navigator.clipboard.writeText(keywords)
        .then(() => {
          alert('Keywords copied to clipboard!');
        })
        .catch(err => {
          console.error('Failed to copy: ', err);
        });
    };
    

    return {
      keyword,
      relatedPhrases,
      loading,
      error,
      fetchKeywords,
      getIntentClass,
      sortTable,
      sortedPhrases,
      copyToClipboard,performSearch,
      selectedSpeed
      
    };
  },
};
</script>

<template>
  <div class="max-w-6xl mx-auto p-6">
    <h1 class="text-2xl font-bold mb-4 py-3">
      <img src="../src/assets/favicon-32x32.png" class="inline"/>
      Keyword Explorer
      <small>v1.0</small>
    </h1>

    <input
      v-model="keyword"
      placeholder="Enter a keyword"
      class="border border-gray-300 p-2 rounded w-full mb-2"
    />
    <select v-model="selectedSpeed" class="border border-gray-300 p-2 rounded mb-2 hidden">
      <option value="fast">Fast (100 tokens)</option>
      <option value="medium">Medium (300 tokens)</option>
      <option value="slow">Slow (500 tokens)</option>
    </select>
    <button
      @click="fetchKeywords"
      class="bg-blue-500 text-white p-2 rounded hover:bg-blue-600"
    >
      Generate Keywords
    </button>
    <div v-if="loading" class="mt-2">Loading...</div>
    <div v-if="error" class="text-red-500 mt-2">{{ error }}</div>


    <div v-if="relatedPhrases.length >= 0" class="mt-4">
      <span class="font-semibold">
        {{ relatedPhrases.length }} result{{ relatedPhrases.length !== 1 ? 's' : '' }} found. <a class="text-blue-500" href="/pro">Unlock more features with Pro</a>
      </span>
    </div>

    <button
      @click="copyToClipboard"
      class="bg-green-500 text-white p-2 rounded hover:bg-green-600 mt-4"
    >
      Copy Keywords to Clipboard
    </button>

    <table v-if="relatedPhrases.length" class="min-w-full mt-4 border border-gray-300">
      <thead>
        <tr class="bg-gray-200">
          <th @click="sortTable('keyword')" class="cursor-pointer p-2">Keyword</th>
          <th @click="sortTable('intent')" class="cursor-pointer p-2">Intent</th>
          <th @click="sortTable('volume')" class="cursor-pointer p-2">Volume</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(phrase, index) in sortedPhrases" :key="index" class="border-b">
          <td class="p-2 cursor-pointer text-blue-500 hover:underline" @click="performSearch(phrase.keyword)">
            {{ phrase.keyword }}
          </td>
          <td class="p-2">
            <span :class="getIntentClass(phrase.intent) + ' text-white px-2 py-1 rounded'">
              {{ phrase.intent }}
            </span>
          </td>
          <td class="p-2">
            {{ phrase.volume.data[0].vol }}
          </td>
        </tr>
      </tbody>
    </table>
  
  </div>
  <footer class="text-center p-10 bg-gray-100 static w-[100%] bottom-0">
    <small>
      Made by <a href="https://diglabs.co.uk" class="font-bold text-blue-400">diglabs.co.uk</a> - Tools for marketers
    </small>
  </footer>
</template>

<style scoped>
/* Add your styles here */
table {
  width: 100%;
  border-collapse: collapse;
}
th {
  cursor: pointer;
}
th:hover {
  background-color: #f0f0f0;
}
.cursor-pointer {
  cursor: pointer;
}
.text-blue-500 {
  color: #3b82f6; /* Tailwind blue-500 */
}
.hover\:underline:hover {
  text-decoration: underline;
}
</style>
