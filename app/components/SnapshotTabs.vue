<script setup lang="ts">
import { assumeUtxoGroups, fastSyncGroups, utreexoGroups } from '~/data/index'

const props = defineProps<{
  activeTab: 'assumeutxo' | 'snapshots' | 'utreexo'
}>()

const emit = defineEmits<{
  switchTab: [tab: 'assumeutxo' | 'snapshots' | 'utreexo']
}>()

// ── Copy magnet ───────────────────────────────────────────────────────────────

const copiedMagnet = ref<string | null>(null)

async function copyMagnet(magnet: string) {
  await navigator.clipboard.writeText(magnet)
  copiedMagnet.value = magnet
  setTimeout(() => { copiedMagnet.value = null }, 1500)
}

// ── Copy utreexo snapshot ─────────────────────────────────────────────────────

const copiedSnapshot = ref<string | null>(null)

async function copySnapshot(snapshot: string) {
  await navigator.clipboard.writeText(snapshot)
  copiedSnapshot.value = snapshot
  setTimeout(() => { copiedSnapshot.value = null }, 1500)
}

// ── Utreexo snapshot QR code ──────────────────────────────────────────────────

const qr = ref<{ name: string; dataUrl: string } | null>(null)

async function showQr(name: string, snapshot: string) {
  const QRCode = await import('qrcode')
  // The string is uppercase bech32, so it fits the compact alphanumeric QR mode.
  const dataUrl = await QRCode.toDataURL(snapshot, {
    errorCorrectionLevel: 'L',
    margin: 2,
    scale: 6,
  })
  qr.value = { name, dataUrl }
}

function closeQr() {
  qr.value = null
}

function onKeydown(e: KeyboardEvent) {
  if (e.key === 'Escape') closeQr()
}

onMounted(() => window.addEventListener('keydown', onKeydown))
onBeforeUnmount(() => window.removeEventListener('keydown', onKeydown))
</script>

<template>
  <!-- Tab bar -->
  <div class="tab-bar" role="tablist">
    <button
      class="tab-btn"
      :class="{ active: activeTab === 'assumeutxo' }"
      role="tab"
      :aria-selected="activeTab === 'assumeutxo'"
      aria-controls="tab-assumeutxo"
      @click="emit('switchTab', 'assumeutxo')"
    >
      Assume UTXO
    </button>
    <button
      class="tab-btn"
      :class="{ active: activeTab === 'snapshots' }"
      role="tab"
      :aria-selected="activeTab === 'snapshots'"
      aria-controls="tab-snapshots"
      @click="emit('switchTab', 'snapshots')"
    >
      FastSync
    </button>
    <button
      class="tab-btn"
      :class="{ active: activeTab === 'utreexo' }"
      role="tab"
      :aria-selected="activeTab === 'utreexo'"
      aria-controls="tab-utreexo"
      @click="emit('switchTab', 'utreexo')"
    >
      Utreexo
    </button>
  </div>

  <!-- Assume UTXO panel -->
  <div v-if="activeTab === 'assumeutxo'" id="tab-assumeutxo" class="tab-panel" role="tabpanel">
    <table class="snap-table" cellpadding="0" cellspacing="0">
      <thead>
        <tr>
          <th>File</th>
          <th>Height</th>
          <th>Best Block</th>
          <th>Hash Serialized</th>
          <th>Chain Txs</th>
          <th>Supply</th>
          <th>Download</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="group in assumeUtxoGroups" :key="group.network">
          <tr class="net-row">
            <td colspan="7"><span :class="['nbadge', group.network]">{{ group.network.toUpperCase() }}</span></td>
          </tr>
          <tr v-for="row in group.entries" :key="row.file">
            <td>
              <div class="fname">{{ row.file }}</div>
              <div class="fsince">since <a :href="row.since.url" rel="nofollow" target="_blank">{{ row.since.label }}</a></div>
            </td>
            <td class="height">
              <a :href="row.blockUrl" target="_blank">{{ row.heightDisplay }}</a>
            </td>
            <td class="hash">{{ row.bestBlock[0] }}<br>{{ row.bestBlock[1] }}</td>
            <td class="hash">{{ row.hashSerialized[0] }}<br>{{ row.hashSerialized[1] }}</td>
            <td>{{ row.chainTxs }}</td>
            <td>{{ row.supply }}</td>
            <td>
              <div style="display:flex;flex-direction:column;gap:4px">
                <a
                  class="dl-btn"
                  :href="row.magnet"
                  target="_blank"
                  @contextmenu.prevent="copyMagnet(row.magnet)"
                >
                  <svg v-if="copiedMagnet !== row.magnet" viewBox="0 0 16 16"><path d="M8 2v9M4 8l4 4 4-4M2 14h12"/></svg>
                  {{ copiedMagnet === row.magnet ? 'Copied!' : 'Torrent' }}
                </a>
                <a class="dl-btn dl-btn-direct" :href="row.url" target="_blank">
                  <svg viewBox="0 0 16 16"><path d="M8 2v9M4 8l4 4 4-4M2 14h12"/></svg>
                  Direct
                </a>
              </div>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>

  <!-- FastSync panel -->
  <div v-if="activeTab === 'snapshots'" id="tab-snapshots" class="tab-panel" role="tabpanel">
    <div class="warning-panel">
      <strong>Warning:</strong> FastSync snapshots require you to trust the snapshot provider.
      This is a <a href="https://docs.btcpayserver.org/Docker/fastsync/#what-are-the-downsides-of-fast-sync" target="_blank" rel="noopener noreferrer">less secure method</a>.
      The recommended approach is to use
      <button style="background:none;border:none;color:var(--danger);text-decoration:underline;cursor:pointer;padding:0;font:inherit;" @click="emit('switchTab', 'assumeutxo')">Assume UTXO</button>
      files, which are verified by your Bitcoin Core node against a hash hardcoded in the source code.
    </div>

    <table class="snap-table" cellpadding="0" cellspacing="0">
      <thead>
        <tr>
          <th>File</th>
          <th>Height</th>
          <th>Best Block</th>
          <th>MuHash</th>
          <th>TxOuts</th>
          <th>Supply</th>
          <th>Download</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="group in fastSyncGroups" :key="group.network">
          <tr class="net-row">
            <td colspan="7"><span :class="['nbadge', group.network]">{{ group.network.toUpperCase() }}</span></td>
          </tr>
          <tr v-for="row in group.entries" :key="row.file">
            <td>
              <div class="fname">{{ row.file }}</div>
              <div class="fsince">{{ row.note }}</div>
            </td>
            <td class="height">
              <a :href="row.blockUrl" target="_blank">{{ row.heightDisplay }}</a>
            </td>
            <td class="hash">{{ row.bestBlock[0] }}<br>{{ row.bestBlock[1] }}</td>
            <td class="hash">{{ row.muhash[0] }}<br>{{ row.muhash[1] }}</td>
            <td>{{ row.txouts }}</td>
            <td>{{ row.supply }}</td>
            <td>
              <div style="display:flex;flex-direction:column;gap:4px">
                <a
                  class="dl-btn"
                  :href="row.magnet"
                  target="_blank"
                  @contextmenu.prevent="copyMagnet(row.magnet)"
                >
                  <svg v-if="copiedMagnet !== row.magnet" viewBox="0 0 16 16"><path d="M8 2v9M4 8l4 4 4-4M2 14h12"/></svg>
                  {{ copiedMagnet === row.magnet ? 'Copied!' : 'Torrent' }}
                </a>
                <a class="dl-btn dl-btn-direct" :href="row.url" target="_blank">
                  <svg viewBox="0 0 16 16"><path d="M8 2v9M4 8l4 4 4-4M2 14h12"/></svg>
                  Direct
                </a>
              </div>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>

  <!-- Utreexo (Floresta / Mandacaru) panel -->
  <div v-if="activeTab === 'utreexo'" id="tab-utreexo" class="tab-panel" role="tabpanel">
    <div class="warning-panel">
      <strong>Warning:</strong> a utreexo snapshot is a short string holding the accumulator roots at a given block.
      Nothing in the string proves the roots are correct, so importing it means trusting the provider
      until your node validates the chain by itself.
    </div>

    <table class="snap-table" cellpadding="0" cellspacing="0">
      <thead>
        <tr>
          <th>Snapshot</th>
          <th>Height</th>
          <th>Best Block</th>
          <th>Block Date</th>
          <th>Leaves</th>
          <th>Roots</th>
          <th>Copy</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="group in utreexoGroups" :key="group.network">
          <tr class="net-row">
            <td colspan="7"><span :class="['nbadge', group.network]">{{ group.network.toUpperCase() }}</span></td>
          </tr>
          <tr v-for="row in group.entries" :key="row.name">
            <td>
              <div class="fname">{{ row.name }}</div>
              <div class="fsince fsince-wrap">{{ row.note }}</div>
            </td>
            <td class="height">
              <a :href="row.blockUrl" target="_blank">{{ row.heightDisplay }}</a>
            </td>
            <td class="hash">{{ row.bestBlock[0] }}<br>{{ row.bestBlock[1] }}</td>
            <td>{{ row.blockDate }}</td>
            <td>{{ row.leaves }}</td>
            <td>{{ row.roots }}</td>
            <td>
              <div style="display:flex;flex-direction:column;gap:4px">
                <button class="dl-btn dl-btn-wide" type="button" @click="copySnapshot(row.snapshot)">
                  {{ copiedSnapshot === row.snapshot ? 'Copied!' : 'Copy' }}
                </button>
                <button class="dl-btn dl-btn-direct dl-btn-wide" type="button" @click="showQr(row.name, row.snapshot)">
                  Show QR code
                </button>
              </div>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>

  <!-- Utreexo QR code modal -->
  <Teleport to="body">
    <div v-if="qr" class="qr-overlay" @click.self="closeQr">
      <div class="qr-modal" role="dialog" aria-modal="true" :aria-label="`QR code for ${qr.name}`">
        <button class="qr-close" type="button" aria-label="Close" @click="closeQr">×</button>
        <div class="qr-title">{{ qr.name }}</div>
        <img class="qr-img" :src="qr.dataUrl" :alt="`QR code for ${qr.name}`">
      </div>
    </div>
  </Teleport>
</template>
