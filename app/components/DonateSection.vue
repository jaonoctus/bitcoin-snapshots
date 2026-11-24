<script setup lang="ts">
const dismissed = ref(false)
</script>

<template>
  <aside v-if="!dismissed" class="donate-float" aria-label="Support this project">
    <p class="donate-copy">
      <span class="donate-warn" aria-hidden="true">⚠</span>
      <span>
        <strong>I personally pay $12/month to keep these snapshots free.</strong>
        <span class="donate-detail">Even $1 counts toward keeping them available.</span>
      </span>
    </p>
    <a class="ln-addr" href="lightning:snapshots@ln.jaonoctus.dev">
      <span class="ln-icon">⚡</span>
      snapshots@ln.jaonoctus.dev
    </a>
    <button class="donate-close" type="button" aria-label="Dismiss" @click="dismissed = true">×</button>
  </aside>
</template>
