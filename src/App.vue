<script setup lang="ts">
  import { ref } from 'vue';
  import { 
    default as DocumentViewer,
    type Highlight
  } from './components/DocumentViewer.vue';

  const pdfURL = ref<string>();
  const highlights = ref<Highlight[]>();

  pdfURL.value = 'input.pdf';

  highlights.value = [{
    page: 1,
    pageBbox: '0 0 1275 1651',
    highlightBbox: '169 165 361 191',
  }, {
    page: 1,
    pageBbox: '0 0 1275 1651',
    highlightBbox: '153 1185 276 1201',
  }, {
    page: 1,
    pageBbox: '0 0 1275 1651',
    highlightBbox: '371 164 575 194'
  }, {
    page: 1,
    pageBbox: '0 0 1275 1651',
    highlightBbox: '586 165 662 198'
  }, {
    page: 1,
    pageBbox: '0 0 1275 1651',
    highlightBbox: '674 164 894 198'
  }, {
    page: 1,
    pageBbox: '0 0 1275 1651',
    highlightBbox: '904 164 950 191'
  }, {
    page: 1,
    pageBbox: '0 0 1275 1651',
    highlightBbox: '960 164 1103 198'
  }];
  
</script>

<template>
  <DocumentViewer :src="pdfURL" :highlights="highlights" />
</template>