<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>
}

const isOnline = ref(navigator.onLine)
const showInstallPrompt = ref(false)
const showUpdatePrompt = ref(false)
let deferredPrompt: BeforeInstallPromptEvent | null = null

// PWA Install Handler
const handleInstall = async () => {
  if (deferredPrompt) {
    deferredPrompt.prompt()
    const { outcome } = await deferredPrompt.userChoice
    console.log(`User response to the install prompt: ${outcome}`)
    deferredPrompt = null
    showInstallPrompt.value = false
  }
}

// PWA Update Handler
const handleUpdate = () => {
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.getRegistration().then((registration) => {
      if (registration && registration.waiting) {
        registration.waiting.postMessage({ type: 'SKIP_WAITING' })
        window.location.reload()
      }
    })
  }
}

// Online/Offline Status
const updateOnlineStatus = () => {
  isOnline.value = navigator.onLine
}

onMounted(() => {
  // Listen for PWA install prompt
  window.addEventListener('beforeinstallprompt', (e) => {
    e.preventDefault()
    deferredPrompt = e as BeforeInstallPromptEvent
    showInstallPrompt.value = true
  })

  // Listen for PWA update
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.addEventListener('controllerchange', () => {
      showUpdatePrompt.value = true
    })
  }

  // Listen for online/offline events
  window.addEventListener('online', updateOnlineStatus)
  window.addEventListener('offline', updateOnlineStatus)
})
</script>

<template>
  <div class="app">
    <!-- PWA Status Bar -->
    <div class="pwa-status">
      <div class="status-item" :class="{ offline: !isOnline }">
        <span class="status-dot"></span>
        {{ isOnline ? 'Online' : 'Offline' }}
      </div>
    </div>

    <!-- PWA Install Prompt -->
    <div v-if="showInstallPrompt" class="pwa-prompt install-prompt">
      <div class="prompt-content">
        <h3>📱 Install App</h3>
        <p>Install aplikasi ini untuk pengalaman yang lebih baik!</p>
        <div class="prompt-actions">
          <button @click="handleInstall" class="btn-primary">Install</button>
          <button @click="showInstallPrompt = false" class="btn-secondary">Nanti</button>
        </div>
      </div>
    </div>

    <!-- PWA Update Prompt -->
    <div v-if="showUpdatePrompt" class="pwa-prompt update-prompt">
      <div class="prompt-content">
        <h3>🔄 Update Tersedia</h3>
        <p>Versi baru aplikasi tersedia. Update sekarang?</p>
        <div class="prompt-actions">
          <button @click="handleUpdate" class="btn-primary">Update</button>
          <button @click="showUpdatePrompt = false" class="btn-secondary">Nanti</button>
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <main class="main-content">
      <div class="hero">
        <h1>🚀 PWA Vue App</h1>
        <p class="subtitle">Progressive Web App dengan Vue.js</p>
        
        <div class="features">
          <div class="feature-card">
            <h3>📱 Installable</h3>
            <p>Dapat diinstall seperti aplikasi native</p>
          </div>
          
          <div class="feature-card">
            <h3>⚡ Fast</h3>
            <p>Loading cepat dengan service worker caching</p>
          </div>
          
          <div class="feature-card">
            <h3>🔄 Offline</h3>
            <p>Bekerja bahkan tanpa koneksi internet</p>
          </div>
        </div>
        
        <p class="docs-link">
          Pelajari lebih lanjut di 
          <a href="https://vuejs.org/" target="_blank" rel="noopener">vuejs.org</a>
        </p>
      </div>
    </main>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.pwa-status {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
}

.status-item {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 500;
}

.status-item.offline {
  background: rgba(255, 107, 107, 0.2);
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #4ade80;
}

.status-item.offline .status-dot {
  background: #f87171;
}

.pwa-prompt {
  position: fixed;
  bottom: 20px;
  left: 20px;
  right: 20px;
  max-width: 400px;
  margin: 0 auto;
  z-index: 1000;
}

.prompt-content {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  color: #333;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
}

.prompt-content h3 {
  margin: 0 0 8px 0;
  font-size: 18px;
}

.prompt-content p {
  margin: 0 0 16px 0;
  color: #666;
}

.prompt-actions {
  display: flex;
  gap: 12px;
}

.btn-primary, .btn-secondary {
  flex: 1;
  padding: 10px 16px;
  border: none;
  border-radius: 8px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-primary {
  background: #4DBA87;
  color: white;
}

.btn-primary:hover {
  background: #42a076;
  transform: translateY(-1px);
}

.btn-secondary {
  background: #e5e7eb;
  color: #374151;
}

.btn-secondary:hover {
  background: #d1d5db;
}

.main-content {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 20px;
}

.hero {
  text-align: center;
  max-width: 800px;
}

.hero h1 {
  font-size: 3.5rem;
  margin: 0 0 16px 0;
  font-weight: 700;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.subtitle {
  font-size: 1.25rem;
  margin: 0 0 48px 0;
  opacity: 0.9;
}

.features {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 24px;
  margin: 48px 0;
}

.feature-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  padding: 24px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: transform 0.2s;
}

.feature-card:hover {
  transform: translateY(-4px);
}

.feature-card h3 {
  font-size: 1.25rem;
  margin: 0 0 12px 0;
}

.feature-card p {
  margin: 0;
  opacity: 0.8;
  line-height: 1.5;
}

.docs-link {
  margin-top: 48px;
  font-size: 1.1rem;
}

.docs-link a {
  color: #4ade80;
  text-decoration: none;
  font-weight: 500;
}

.docs-link a:hover {
  text-decoration: underline;
}

@media (max-width: 768px) {
  .hero h1 {
    font-size: 2.5rem;
  }
  
  .features {
    grid-template-columns: 1fr;
  }
  
  .pwa-prompt {
    left: 10px;
    right: 10px;
  }
}
</style>
