<script setup lang="ts">
useHead({
  title: 'Bitcoin UTXO Snapshots - Fast IBD via Assume UTXO & FastSync Torrents',
  meta: [
    { name: 'description', content: 'Download Bitcoin UTXO snapshots for fast initial block download. Verified Assume UTXO .dat files hardcoded in Bitcoin Core, or FastSync .tar.zst torrents for mainnet, testnet3, testnet4, and signet.' },
    { property: 'og:type', content: 'website' },
    { property: 'og:url', content: 'https://bitcoin-snapshots.jaonoctus.dev/' },
    { property: 'og:title', content: 'Bitcoin UTXO Snapshots - Assume UTXO & FastSync Torrents' },
    { property: 'og:description', content: 'Download Bitcoin UTXO snapshots for fast initial block download. Verified Assume UTXO .dat files hardcoded in Bitcoin Core, or FastSync .tar.zst torrents for mainnet, testnet3, testnet4, and signet.' },
    { property: 'og:image', content: 'https://bitcoin-snapshots.jaonoctus.dev/censorship.jpg' },
    { name: 'twitter:card', content: 'summary_large_image' },
    { name: 'twitter:image', content: 'https://bitcoin-snapshots.jaonoctus.dev/censorship.jpg' },
    { name: 'twitter:title', content: 'Bitcoin UTXO Snapshots - Assume UTXO & FastSync Torrents' },
    { name: 'twitter:description', content: 'Download Bitcoin UTXO snapshots for fast initial block download. Verified Assume UTXO .dat files hardcoded in Bitcoin Core, or FastSync .tar.zst torrents for mainnet, testnet3, testnet4, and signet.' },
  ],
  link: [
    { rel: 'canonical', href: 'https://bitcoin-snapshots.jaonoctus.dev/' },
  ],
})

const activeTab = ref<'assumeutxo' | 'snapshots' | 'utreexo'>('assumeutxo')
</script>

<template>
  <DonateSection />
  <SiteHeader />

  <main class="site-main">
    <section class="supporters">
      <h3>Supporters</h3>
      <p>
        Shout out to
        <a href="https://x.com/rusty_twit" target="_blank" rel="noopener noreferrer">Rusty Russell</a>
        and
        <a href="https://x.com/evankaloudis" target="_blank" rel="noopener noreferrer">Evan Kaloudis</a>
        for donating and keeping this project running in previous years.
      </p>
    </section>

    <div class="sec-head">
      <h2>Browse Archives</h2>
      <div class="rule"></div>
    </div>

    <SnapshotTabs :active-tab="activeTab" @switch-tab="activeTab = $event" />
    <InfoCards :active-tab="activeTab" />
  </main>

  <footer class="site-footer">
    made by <a href="https://jaonoctus.dev" target="_blank" rel="noopener noreferrer">jaonoctus</a> with <span class="love">love</span>
    | <a href="https://github.com/jaonoctus/bitcoin-snapshots" target="_blank" rel="noopener noreferrer">github</a>
  </footer>
</template>
