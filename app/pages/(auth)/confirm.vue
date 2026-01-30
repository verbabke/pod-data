<template>
  <div class="ma-auto d-flex align-center">
    <v-progress-circular indeterminate color="primary" />
    <span class="ml-4">Verifying...</span>
  </div>
</template>

<script setup lang="ts">
useHead({
  title: 'Confirming...',
})

const user = useSupabaseUser()
const client = useSupabaseClient()
const route = useRoute()

watch(
  user,
  () => {
    if (user.value) {
      // If the user is currently anonymous and there are auth params (code/hash) in the URL,
      // we need to wait for the session to update locally (convert from anonymous to authenticated)
      // before redirecting. The Supabase client handles the code exchange, and user.value will update.
      const hasAuthParams = route.query.code || route.hash || route.query.token_hash
      if (user.value.is_anonymous && hasAuthParams) {
        return
      }

      const redirectPath = (route.query.redirect as string) || '/'
      navigateTo(redirectPath)
    }
  },
  { immediate: true }
)

onMounted(async () => {
  // Check for error in URL (Supabase redirects here with error params if link is invalid)
  const errorDescription = route.query.error_description as string
  const error = route.query.error as string

  if (error || errorDescription) {
    // Redirect to login with the error message
    return navigateTo({
      path: '/login',
      query: {
        ...route.query,
        error_description: errorDescription || error || 'An error occurred during verification.',
      },
    })
  }

  // Wait for the Supabase client to handle the PKCE code exchange automatically.
  // If anonymous and code is present, the client should upgrade the session.
  // Set a timeout to force a check if it stalls.
  const code = route.query.code
  if (user.value?.is_anonymous && code) {
    // Listen for auth state changes to detect when the anonymous user is upgraded
    const {
      data: { subscription },
    } = client.auth.onAuthStateChange((event, session) => {
      if (session && !session.user.is_anonymous) {
        // User is no longer anonymous, the watcher will handle the redirect
        subscription.unsubscribe()
      }
    })

    // Fallback: If nothing happens after a short delay, try to refresh the session
    setTimeout(async () => {
      if (user.value?.is_anonymous) {
        await client.auth.refreshSession()
      }
    }, 1500)
  }

  // If user lands here without a code/hash and isn't logged in, redirect to login
  const hasCode = route.query.code || route.hash || route.query.token_hash

  if (!user.value && !hasCode) {
    navigateTo('/login')
  }
})
</script>
