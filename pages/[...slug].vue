<script setup lang="ts">
import { ref, computed } from 'vue';
import { useRoute } from 'vue-router';
import { Content, isPreviewing, fetchOneEntry } from '@builder.io/sdk-vue';

// TODO: enter your public API key
const apiKey = '';

const route = useRoute();
const urlPath = computed(() => {
  const path = Array.isArray(route.params.slug) ? route.params.slug.join('/') : route.params.slug;
  return `/${path}`;
});
const content = ref(null);
const canShowContent = ref(false);

// Fetch the Builder content for the page that matches the URL path
try {
  const response = await fetchOneEntry({
    model: 'page',
    apiKey,
    userAttributes: {
      urlPath: urlPath.value,
    },
  });
  content.value = response;
  canShowContent.value = content.value || isPreviewing();
} catch (error) {
  console.error('Builder content not found:', error);
  if (process.server) {
    // Set the HTTP response status code to 404 if no content is found
    const nuxtApp = useNuxtApp();
    nuxtApp.ssrContext.event.res.statusCode = 404;
  }
}
</script>

<template>
  <div id="home">
    <div>Hello world from your Nuxt 3 project. Below is Builder Content:</div>
    <div v-if="canShowContent">
      <div>
        Page title:
        {{ (content?.data?.title) || 'Unpublished' }}
      </div>
      <Content model="page" :content="content" :api-key="apiKey" />
    </div>
    <div v-else>Content not Found</div>
  </div>
</template>
