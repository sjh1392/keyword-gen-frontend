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

    const fetchKeywords = async () => {

      loading.value = true;
      error.value = '';

      try {

        const response = await fetch(`https://keywordio-d7419b16e33c.herokuapp.com/api/related-searches?keyword=${keyword.value}`);
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


    const sortTable = (key) => {
      sortOrder.value = (sortKey.value === key && sortOrder.value === 'asc') ? 'desc' : 'asc';
      sortKey.value = key;
    };

    const sortedPhrases = computed(() => {
      return [...relatedPhrases.value].sort((a, b) => {
        const modifier = sortOrder.value === 'asc' ? 1 : -1;
        if (sortKey.value === 'volume') {
          return (Number(a.volume) - Number(b.volume)) * modifier; // Compare as numbers
        }
        if (a[sortKey.value] < b[sortKey.value]) return -1 * modifier;
        if (a[sortKey.value] > b[sortKey.value]) return 1 * modifier;
        return 0;
      });
    });

    const filteredPhrases = computed(() => {
      return sortedPhrases.value.filter(phrase => 
        phrase.keyword.toLowerCase().includes(keyword.value.toLowerCase())
      );
    });

    return {
      keyword,
      relatedPhrases,
      loading,
      error,
      fetchKeywords,
      getIntentClass,
      sortTable,
      filteredPhrases,
    };
  },
};
</script>

<template>
  <div class="max-w-6xl mx-auto p-6">
    <h1 class="text-2xl font-bold mb-4">Keyword Generator</h1>
    <input
      v-model="keyword"
      placeholder="Enter a keyword"
      class="border border-gray-300 p-2 rounded w-full mb-2"
    />
    <button
      @click="fetchKeywords"
      class="bg-blue-500 text-white p-2 rounded hover:bg-blue-600"
    >
      Generate Keywords
    </button>
    <div v-if="loading" class="mt-2">Loading...</div>
    <div v-if="error" class="text-red-500 mt-2">{{ error }}</div>


    <div v-if="filteredPhrases.length >= 0" class="mt-4">
      <span class="font-semibold">
        {{ filteredPhrases.length }} result{{ filteredPhrases.length !== 1 ? 's' : '' }} found. <a class="text-blue-500" href="/pro">Unlock more features with Pro</a>
      </span>
    </div>


    <table v-if="relatedPhrases.length" class="min-w-full mt-4 border border-gray-300">
      <thead>
        <tr class="bg-gray-200">
          <th @click="sortTable('keyword')" class="cursor-pointer p-2">Keyword</th>
          <th @click="sortTable('intent')" class="cursor-pointer p-2">Intent</th>
          <th @click="sortTable('volume')" class="cursor-pointer p-2">Volume</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(phrase, index) in filteredPhrases" :key="index" class="border-b">
          <td class="p-2">{{ phrase.keyword }}</td>
          <td class="p-2">
            <span :class="getIntentClass(phrase.intent) + ' text-white px-2 py-1 rounded'">
              {{ phrase.intent }}
            </span>
            </td>
          <td class="p-2">{{ phrase.volume.data[0].vol }}</td>
        </tr>
      </tbody>
    </table>
  
  </div>
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
</style>
