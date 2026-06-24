<script setup lang="ts">
import { ref, onMounted, onUnmounted, type Ref } from 'vue';
import { useNav } from '@slidev/client';

interface ScreenDetails {
  isInternal: boolean;
  availLeft: number;
  availTop: number;
  availWidth: number;
  availHeight: number;
}

const { isPresenter } = useNav();

// Check if Window Management API is supported (synchronous)
const isApiSupported = typeof window !== 'undefined' && 'getScreenDetails' in window;

// State
const secondWindow: Ref<Window | null> = ref(null);
let fullscreenObserver: MutationObserver | null = null;
let keydownHandler: ((e: KeyboardEvent) => void) | null = null;

// Feature detection on mount
onMounted(() => {
  // Detect if this is the second screen window (normal slide view opened from presenter)
  const hasOpener = !!window.opener;
  const isPres = isPresenter.value;

  if (hasOpener && !isPres) {
    // Try to enter fullscreen automatically (only if API is supported)
    if (isApiSupported) {
      setTimeout(async () => {
        try {
          if (document.documentElement.requestFullscreen) {
            await document.documentElement.requestFullscreen();
            console.log('[SecondScreen] Fullscreen entered successfully');
          }
        } catch (err) {
          console.log('[SecondScreen] Direct fullscreen failed, showing prompt:', err);
          createFullscreenPrompt();
        }
      }, 500);
    }
  }

  // Listen for fullscreen changes to remove prompt
  document.addEventListener('fullscreenchange', handleFullscreenChange);

  // Listen for messages from second screen window
  window.addEventListener('message', handlePresenterMessage);
});

onUnmounted(() => {
  document.removeEventListener('fullscreenchange', handleFullscreenChange);
  window.removeEventListener('message', handlePresenterMessage);
  if (keydownHandler) {
    document.removeEventListener('keydown', keydownHandler);
  }
  if (fullscreenObserver) {
    fullscreenObserver.disconnect();
  }
  removeFullscreenPrompt();
});

// Notify parent window when this window is closed
const beforeUnloadHandler = () => {
  if (window.opener && !window.opener.closed) {
    window.opener.postMessage({ type: 'second-screen-closed' }, '*');
  }
};

if (typeof window !== 'undefined') {
  window.addEventListener('beforeunload', beforeUnloadHandler);
}

// Fullscreen prompt management
function createFullscreenPrompt() {
  removeFullscreenPrompt();

  const overlay = document.createElement('div');
  overlay.id = 'second-screen-fullscreen-prompt';
  overlay.style.cssText = `
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.85);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 999999;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  `;

  const content = document.createElement('div');
  content.style.cssText = `
    position: relative;
    background: white;
    padding: 40px;
    border-radius: 12px;
    text-align: center;
    max-width: 400px;
    box-shadow: 0 10px 40px rgba(0,0,0,0.3);
  `;

  content.innerHTML = `
    <button id="close-prompt-btn" style="
      position: absolute;
      top: 12px;
      right: 12px;
      background: transparent;
      border: none;
      font-size: 20px;
      color: #999;
      cursor: pointer;
      line-height: 1;
      padding: 4px 8px;
    ">✕</button>
    <h2 style="margin: 0 0 20px 0; color: #333;">🖥️ Second Screen Mode</h2>
    <p style="margin: 0 0 30px 0; color: #666; line-height: 1.6;">
      Click the button below to enter fullscreen mode for the best presentation experience.
    </p>
    <button id="enter-fullscreen-btn" style="
      background: #3b82f6;
      color: white;
      border: none;
      padding: 12px 32px;
      font-size: 16px;
      border-radius: 6px;
      cursor: pointer;
      font-weight: 600;
      transition: background 0.2s;
    ">
      Enter Fullscreen
    </button>
    <p style="margin: 20px 0 0 0; color: #999; font-size: 12px;">
      Or press F11 / Cmd+Shift+F
    </p>
  `;

  overlay.appendChild(content);
  document.body.appendChild(overlay);

  const btn = document.getElementById('enter-fullscreen-btn');
  if (btn) {
    btn.addEventListener('click', async () => {
      try {
        if (document.documentElement.requestFullscreen) {
          await document.documentElement.requestFullscreen();
          removeFullscreenPrompt();
        }
      } catch (err) {
        console.log('[SecondScreen] Fullscreen request failed:', err);
      }
    });

    btn.addEventListener('mouseenter', () => { btn.style.background = '#2563eb'; });
    btn.addEventListener('mouseleave', () => { btn.style.background = '#3b82f6'; });
  }

  const closeBtn = document.getElementById('close-prompt-btn');
  if (closeBtn) {
    closeBtn.addEventListener('click', () => {
      removeFullscreenPrompt();
    });
    closeBtn.addEventListener('mouseenter', () => { closeBtn.style.color = '#333'; });
    closeBtn.addEventListener('mouseleave', () => { closeBtn.style.color = '#999'; });
  }

  // Keyboard shortcuts: Enter for fullscreen, Escape to close
  keydownHandler = (e: KeyboardEvent) => {
    if (e.key === 'Enter') {
      const btn = document.getElementById('enter-fullscreen-btn');
      if (btn) btn.click();
    } else if (e.key === 'Escape') {
      removeFullscreenPrompt();
    }
  };
  document.addEventListener('keydown', keydownHandler);

  // Clean up keyboard listener when prompt is removed
  fullscreenObserver = new MutationObserver(() => {
    if (!document.getElementById('second-screen-fullscreen-prompt')) {
      if (keydownHandler) {
        document.removeEventListener('keydown', keydownHandler);
        keydownHandler = null;
      }
      if (fullscreenObserver) {
        fullscreenObserver.disconnect();
        fullscreenObserver = null;
      }
    }
  });
  fullscreenObserver.observe(document.body, { childList: true });
}

function removeFullscreenPrompt() {
  const overlay = document.getElementById('second-screen-fullscreen-prompt');
  if (overlay) {
    overlay.remove();
  }
}

function handleFullscreenChange() {
  if (document.fullscreenElement) {
    removeFullscreenPrompt();
  }
}

// Handle messages from second screen window (presenter only)
function handlePresenterMessage(event: MessageEvent) {
  if (event.data?.type === 'second-screen-closed') {
    secondWindow.value = null;
  }
}

// Open presentation on second screen
async function openOnSecondScreen() {
  if (!isApiSupported) {
    alert('Window Management API not supported. Please use Chrome 100+ or Edge 100+.');
    return;
  }

  try {
    const screenDetails = await (window as any).getScreenDetails();
    const currentScreen = screenDetails.currentScreen;

    // Find the screen where the presenter is NOT running
    const targetScreen = screenDetails.screens.find((s: ScreenDetails) => s !== currentScreen);

    if (!targetScreen) {
      alert('No second screen detected.');
      return;
    }

    // Build URL for normal slide view (not presenter)
    const currentUrl = window.location.href;
    const slideUrl = currentUrl.replace('/presenter/', '/');

    // Open window on the other screen (where presenter is NOT running)
    const windowFeatures = `left=${targetScreen.availLeft},top=${targetScreen.availTop},width=${targetScreen.availWidth},height=${targetScreen.availHeight}`;
    secondWindow.value = window.open(slideUrl, 'slidev-second-screen', windowFeatures);

    if (!secondWindow.value) {
      alert('Popup blocked. Please allow popups for this site.');
      return;
    }

  } catch (error) {
    console.error('Failed to open on second screen:', error);
    const message = error instanceof Error ? error.message : String(error);
    alert('Failed to open on second screen: ' + message);
  }
}

// Close second window
function closeSecondWindow() {
  if (secondWindow.value && !secondWindow.value.closed) {
    secondWindow.value.close();
  }
  secondWindow.value = null;
}
</script>

<template>
      <!-- Open on Second Screen Button -->
      <div class="w-1px opacity-10 bg-current m-1 lg:m-2" v-if="isPresenter"></div>
      <button
        class="slidev-icon-btn"
        :class="{ 'opacity-50': !isApiSupported || secondWindow === null }"
        :title="!isApiSupported ? 'Not supported in this browser. Use Chrome 100+ or Edge 100+.' : (secondWindow !== null ? 'Close second screen' : 'Open presentation on second screen')"
        :disabled="!isApiSupported"
        @click="secondWindow !== null ? closeSecondWindow() : openOnSecondScreen()"
        v-if="isPresenter"
      >
        <span class="sr-only">Second Screen</span>
        <svg v-if="secondWindow === null" class="btn-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect>
          <line x1="8" y1="21" x2="16" y2="21"></line>
          <line x1="12" y1="17" x2="12" y2="21"></line>
        </svg>
        <svg v-else class="btn-icon" viewBox="0 0 24 24" fill="currentColor">
          <rect x="2" y="3" width="20" height="14" rx="2" ry="2"></rect>
          <line x1="8" y1="21" x2="16" y2="21" stroke="currentColor" stroke-width="2"></line>
          <line x1="12" y1="17" x2="12" y2="21" stroke="currentColor" stroke-width="2"></line>
        </svg>
      </button>
</template>

<style scoped>
.btn-icon {
  width: 20px;
  height: 20px;
}

button.slidev-icon-btn:disabled {
  cursor: default;
}
</style>
