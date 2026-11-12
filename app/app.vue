<script setup lang="ts">
const title = 'Pod Data'
const description =
  "Pod Data is an unofficial checklist for tracking your completion progress in NieR: Automata. Mark various items, quests, and achievements as completed, to help keep track of what you've done and what still needs to be accomplished."

useSeoMeta({
  title: title,
  description: description,
  ogTitle: title,
  ogDescription: description,
  ogType: 'website',
  twitterTitle: title,
  twitterDescription: description,
  twitterCard: 'summary_large_image',
})

useHead({
  link: [
    {
      rel: 'icon',
      type: 'image/svg+xml',
      href: '/img/robot-primary.svg',
    },
  ],
})

defineOgImage('PodData')
</script>

<template>
  <v-app>
    <NuxtLayout>
      <!-- Main page -->
      <NuxtPage />
    </NuxtLayout>
  </v-app>
</template>
