<script setup lang="ts">
import { ref, computed } from 'vue';
import { 
  Lock, 
  Unlock, 
  Copy, 
  Check, 
  Trash2, 
  ShieldCheck, 
  FileText, 
  X, 
  Sparkles,
  Info,
  CheckCircle2,
  AlertTriangle,
  ClipboardList,
  Sun,
  Moon,
  Loader2
} from 'lucide-vue-next';

interface Toast {
  id: number;
  message: string;
  type: 'success' | 'error' | 'info';
}

interface ConversionHistoryItem {
  id: string;
  type: 'encrypt' | 'decrypt';
  input: string;
  output: string;
  timestamp: string;
  keyLabel: string;
}

const DEFAULT_FIXED_KEY = 'SECURE_APP_VUE_2026_KEY';

// State
const activeTab      = ref<'encrypt' | 'decrypt'>('encrypt');
const isDarkMode     = ref(true);
const customKeyInput = ref('');
const inputText      = ref('');
const resultText     = ref('');
const isCopied       = ref(false);
const isLoading      = ref(false);   // ← lazy loading state
const history        = ref<ConversionHistoryItem[]>([]);
const toasts         = ref<Toast[]>([]);
let nextToastId = 1;

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value;
  showToast(`Switched to ${isDarkMode.value ? 'Dark' : 'Light'} theme`, 'info');
};

// Toast Handler
const showToast = (message: string, type: 'success' | 'error' | 'info' = 'success') => {
  const id = nextToastId++;
  toasts.value.unshift({ id, message, type });
  if (toasts.value.length > 4) toasts.value.pop();
  setTimeout(() => removeToast(id), 3500);
};

const removeToast = (id: number) => {
  toasts.value = toasts.value.filter(t => t.id !== id);
};

// ── Main action with lazy loading ────────────────────────────
const handleConvert = async () => {
  if (!inputText.value.trim()) {
    showToast('Please enter text to process', 'error');
    return;
  }

  const effectiveKey = customKeyInput.value.trim() || DEFAULT_FIXED_KEY;

  isLoading.value = true;   // START loading

  try {
    if (activeTab.value === 'encrypt') {
      const response = await fetch(
        'https://spring-api-decrypt-v1-0-0.onrender.com/api/v1/main/encrypt',
        {
          method:  'POST',
          headers: { 'Content-Type': 'application/json' },
          body:    JSON.stringify({ encryptText: inputText.value }),
        }
      );

      if (!response.ok) throw new Error(`Server error: ${response.status}`);

      const json       = await response.json();
      resultText.value = json.data;
      addToHistory('encrypt', inputText.value, resultText.value, effectiveKey);
      showToast('Text encrypted successfully!', 'success');

    } else {
      const response = await fetch(
        'https://spring-api-decrypt-v1-0-0.onrender.com/api/v1/main/decrypt',
        {
          method:  'POST',
          headers: { 'Content-Type': 'application/json' },
          body:    JSON.stringify({ encryptText: inputText.value }),
        }
      );

      if (!response.ok) throw new Error(`Server error: ${response.status}`);

      const json       = await response.json();
      resultText.value = json.data;
      addToHistory('decrypt', inputText.value, resultText.value, effectiveKey);
      showToast('Text decrypted successfully!', 'success');
    }
  } catch (err: any) {
    showToast(err.message ?? 'Request failed. Please try again.', 'error');
  } finally {
    isLoading.value = false;   // STOP loading (always)
  }
};

const addToHistory = (
  type: 'encrypt' | 'decrypt',
  input: string,
  output: string,
  keyUsed: string
) => {
  const time     = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', second: '2-digit' });
  const keyLabel = keyUsed === DEFAULT_FIXED_KEY ? 'Fixed Code Key' : keyUsed;
  history.value.unshift({
    id: Math.random().toString(36).substring(2, 9),
    type,
    input: input.length > 50 ? input.substring(0, 47) + '...' : input,
    output,
    timestamp: time,
    keyLabel,
  });
  if (history.value.length > 8) history.value.pop();
};

const copyResult = async () => {
  if (!resultText.value) return;
  try {
    await navigator.clipboard.writeText(resultText.value);
    isCopied.value = true;
    showToast('Copied to clipboard!', 'success');
    setTimeout(() => { isCopied.value = false; }, 2000);
  } catch {
    showToast('Failed to copy text', 'error');
  }
};

const clearAll = () => {
  inputText.value  = '';
  resultText.value = '';
  showToast('Cleared input & result', 'info');
};

const switchTab = (tab: 'encrypt' | 'decrypt') => {
  activeTab.value  = tab;
  resultText.value = '';
};

const loadSample = () => {
  activeTab.value  = 'encrypt';
  inputText.value  = 'HELLO, MY NAME IS DMK.';
  showToast('Loaded sample text for encryption', 'info');
};

const inputStats = computed(() => ({
  chars: inputText.value.length,
  words: inputText.value.trim() ? inputText.value.trim().split(/\s+/).length : 0,
}));

const resultStats = computed(() => ({
  chars: resultText.value.length,
  words: resultText.value.trim() ? resultText.value.trim().split(/\s+/).length : 0,
}));
</script>

<template>
  <div :class="[
    'min-h-screen font-sans flex flex-col justify-between p-4 sm:p-6 lg:p-10 transition-colors duration-300',
    isDarkMode
      ? 'bg-slate-950 text-slate-200 selection:bg-emerald-500/30 selection:text-emerald-300'
      : 'bg-slate-50 text-slate-800 selection:bg-emerald-500/20 selection:text-emerald-700'
  ]">

    <!-- Header -->
    <header :class="['flex justify-between items-center mb-8 sm:mb-12 border-b pb-6 max-w-5xl mx-auto w-full transition-colors', isDarkMode ? 'border-slate-800' : 'border-slate-200']">
      <div>
        <h1 class="text-xl sm:text-2xl font-mono font-bold tracking-tighter text-emerald-600 dark:text-emerald-500 flex items-center gap-2">
          <ShieldCheck class="w-6 h-6 text-emerald-600 dark:text-emerald-500" />
          <span>CYPHER_VAULT.v1</span>
        </h1>
        <p :class="['text-[10px] sm:text-xs mt-1 uppercase tracking-widest font-mono', isDarkMode ? 'text-slate-500' : 'text-slate-400']">
          Secure Payload Processor & Text Converter
        </p>
      </div>

      <button
        @click="toggleTheme"
        :class="[
          'flex items-center gap-1.5 px-3 py-1.5 rounded-full text-[11px] font-mono font-semibold border transition-all cursor-pointer shadow-sm',
          isDarkMode
            ? 'bg-slate-900 border-slate-800 text-amber-400 hover:bg-slate-800'
            : 'bg-white border-slate-200 text-slate-700 hover:bg-slate-100'
        ]"
      >
        <Sun v-if="isDarkMode" class="w-3.5 h-3.5" />
        <Moon v-else class="w-3.5 h-3.5 text-indigo-600" />
        <span>{{ isDarkMode ? 'LIGHT MODE' : 'DARK MODE' }}</span>
      </button>
    </header>

    <!-- Main -->
    <main class="max-w-5xl mx-auto w-full flex-1 flex flex-col gap-8">

      <div :class="[
        'w-full border rounded-2xl shadow-2xl overflow-hidden backdrop-blur-sm transition-colors',
        isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200 shadow-slate-200/60'
      ]">

        <!-- Tabs -->
        <div :class="['flex border-b transition-colors', isDarkMode ? 'border-slate-800 bg-slate-950/50' : 'border-slate-200 bg-slate-50']">
          <button
            v-for="tab in (['encrypt', 'decrypt'] as const)"
            :key="tab"
            @click="switchTab(tab)"
            :disabled="isLoading"
            :class="[
              'flex-1 py-4 px-4 text-xs sm:text-sm font-semibold tracking-widest uppercase transition-all flex items-center justify-center gap-2 border-b-2 cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed',
              activeTab === tab
                ? isDarkMode
                  ? 'bg-slate-800/90 text-white border-emerald-500 shadow-inner'
                  : 'bg-white text-slate-900 border-emerald-600 shadow-sm'
                : isDarkMode
                  ? 'text-slate-500 border-transparent hover:bg-slate-800/40 hover:text-slate-300'
                  : 'text-slate-400 border-transparent hover:bg-slate-100 hover:text-slate-700'
            ]"
          >
            <Lock v-if="tab === 'encrypt'" :class="['w-4 h-4', isDarkMode ? 'text-emerald-400' : 'text-emerald-600']" />
            <Unlock v-else :class="['w-4 h-4', isDarkMode ? 'text-emerald-400' : 'text-emerald-600']" />
            <span>{{ tab === 'encrypt' ? 'ENCRYPT_MODE' : 'DECRYPT_MODE' }}</span>
          </button>
        </div>

        <!-- Toolbar -->
        <div :class="[
          'px-6 py-3 border-b flex flex-wrap items-center justify-between gap-3 text-xs font-mono transition-colors',
          isDarkMode ? 'bg-slate-950/60 border-slate-800/60 text-slate-500' : 'bg-slate-50/80 border-slate-200 text-slate-500'
        ]">
          <div class="flex items-center gap-2">
            <Sparkles :class="['w-3.5 h-3.5', isDarkMode ? 'text-emerald-500' : 'text-emerald-600']" />
            <span class="uppercase tracking-wider text-[10px]">PRESETS:</span>
            <button
              @click="loadSample"
              :disabled="isLoading"
              :class="[
                'px-2.5 py-1 rounded transition-colors border text-[11px] cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed',
                isDarkMode
                  ? 'bg-slate-900 hover:bg-slate-800 text-slate-300 border-slate-800'
                  : 'bg-white hover:bg-slate-100 text-slate-700 border-slate-300'
              ]"
            >
              Plaintext Sample
            </button>
          </div>

          <button
            v-if="inputText || resultText"
            @click="clearAll"
            :disabled="isLoading"
            class="text-slate-400 hover:text-rose-500 transition-colors flex items-center gap-1.5 px-2 py-0.5 rounded hover:bg-rose-500/10 text-[11px] cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed"
          >
            <Trash2 class="w-3.5 h-3.5" />
            <span class="uppercase tracking-wider">RESET</span>
          </button>
        </div>

        <div class="p-6 sm:p-8 space-y-6">

          <!-- Input / Output grid -->
          <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">

            <!-- Input -->
            <div class="space-y-2">
              <div class="flex items-center justify-between">
                <label class="block text-[10px] font-bold text-slate-500 uppercase tracking-widest flex items-center gap-1.5">
                  <FileText :class="['w-3.5 h-3.5', isDarkMode ? 'text-emerald-500' : 'text-emerald-600']" />
                  <span>PAYLOAD BODY (INPUT)</span>
                </label>
                <span class="text-[10px] font-mono text-slate-500">
                  {{ inputStats.chars }} CHARS | {{ inputStats.words }} WORDS
                </span>
              </div>
              <textarea
                v-model="inputText"
                rows="8"
                :disabled="isLoading"
                :placeholder="activeTab === 'encrypt' ? 'Paste raw data or message here...' : 'Paste ciphertext payload here...'"
                :class="[
                  'w-full border rounded-lg p-4 text-sm font-mono resize-none focus:outline-none leading-relaxed transition-colors disabled:opacity-50 disabled:cursor-not-allowed',
                  isDarkMode
                    ? 'bg-slate-950 border-slate-800 text-slate-300 focus:border-emerald-500/50 placeholder:text-slate-700'
                    : 'bg-slate-50 border-slate-300 text-slate-800 focus:border-emerald-600 placeholder:text-slate-400'
                ]"
              ></textarea>
            </div>

            <!-- Output -->
            <div class="space-y-2 relative">
              <div class="flex items-center justify-between">
                <label class="block text-[10px] font-bold text-slate-500 uppercase tracking-widest flex items-center gap-1.5">
                  <ShieldCheck :class="['w-3.5 h-3.5', isDarkMode ? 'text-emerald-400' : 'text-emerald-600']" />
                  <span>CONVERTED RESULT (OUTPUT)</span>
                </label>
                <span class="text-[10px] font-mono text-slate-500">
                  {{ resultStats.chars }} CHARS | {{ resultStats.words }} WORDS
                </span>
              </div>

              <textarea
                :value="resultText"
                readonly
                rows="8"
                placeholder="Converted payload will materialize here..."
                :class="[
                  'w-full border rounded-lg p-4 text-sm font-mono focus:outline-none leading-relaxed resize-none transition-colors',
                  isDarkMode
                    ? 'bg-slate-950 border-slate-800 text-emerald-400 placeholder:text-slate-800'
                    : 'bg-slate-50 border-slate-300 text-emerald-700 placeholder:text-slate-400 font-semibold'
                ]"
              ></textarea>

              <!-- Skeleton shimmer while loading -->
              <div
                v-if="isLoading"
                class="absolute inset-0 top-6 rounded-lg overflow-hidden pointer-events-none"
              >
                <div :class="[
                  'w-full h-full rounded-lg animate-pulse space-y-3 p-4',
                  isDarkMode ? 'bg-slate-950' : 'bg-slate-50'
                ]">
                  <div :class="['h-3 rounded w-full', isDarkMode ? 'bg-slate-800' : 'bg-slate-200']"></div>
                  <div :class="['h-3 rounded w-4/5', isDarkMode ? 'bg-slate-800' : 'bg-slate-200']"></div>
                  <div :class="['h-3 rounded w-full', isDarkMode ? 'bg-slate-800' : 'bg-slate-200']"></div>
                  <div :class="['h-3 rounded w-3/5', isDarkMode ? 'bg-slate-800' : 'bg-slate-200']"></div>
                  <div :class="['h-3 rounded w-full', isDarkMode ? 'bg-slate-800' : 'bg-slate-200']"></div>
                  <div :class="['h-3 rounded w-2/3', isDarkMode ? 'bg-slate-800' : 'bg-slate-200']"></div>
                </div>
              </div>

              <!-- Empty state (only when not loading and no result) -->
              <div
                v-if="!resultText && !isLoading"
                class="absolute inset-0 top-6 flex flex-col items-center justify-center pointer-events-none text-slate-700 space-y-2"
              >
                <ClipboardList :class="['w-8 h-8 opacity-30', isDarkMode ? 'text-slate-600' : 'text-slate-400']" />
                <p :class="['text-[11px] font-mono uppercase tracking-widest', isDarkMode ? 'text-slate-600' : 'text-slate-400']">
                  Awaiting execution
                </p>
              </div>
            </div>

          </div>

          <!-- Execute Button -->
          <button
            @click="handleConvert"
            :disabled="isLoading"
            :class="[
              'w-full font-bold py-4 rounded-lg transition-all shadow-lg uppercase tracking-widest text-xs flex items-center justify-center gap-2 cursor-pointer',
              'disabled:opacity-70 disabled:cursor-not-allowed',
              isLoading ? 'active:scale-100' : 'active:scale-[0.99]',
              isDarkMode
                ? 'bg-emerald-600 hover:bg-emerald-500 text-slate-950 shadow-emerald-900/20'
                : 'bg-emerald-600 hover:bg-emerald-700 text-white shadow-emerald-600/20'
            ]"
          >
            <!-- Spinner when loading -->
            <Loader2 v-if="isLoading" class="w-4 h-4 animate-spin" />

            <!-- Normal icon when idle -->
            <template v-else>
              <Lock v-if="activeTab === 'encrypt'" class="w-4 h-4" />
              <Unlock v-else class="w-4 h-4" />
            </template>

            <span>
              {{ isLoading
                  ? (activeTab === 'encrypt' ? 'ENCRYPTING...' : 'DECRYPTING...')
                  : 'EXECUTE CONVERSION'
              }}
            </span>
          </button>

          <!-- Copy button (only shown when result exists and not loading) -->
          <div v-if="resultText && !isLoading" class="pt-2 flex items-center gap-3">
            <button
              @click="copyResult"
              :class="[
                'flex-1 border rounded-lg py-3 px-4 text-xs font-bold font-mono tracking-wider uppercase transition-colors flex items-center justify-center gap-2 cursor-pointer shadow-md',
                isDarkMode
                  ? 'bg-slate-800 hover:bg-slate-700 text-emerald-400 border-slate-700/80'
                  : 'bg-emerald-50 hover:bg-emerald-100 text-emerald-800 border-emerald-200'
              ]"
            >
              <Check v-if="isCopied" class="w-4 h-4 text-emerald-500" />
              <Copy v-else class="w-4 h-4" />
              <span>{{ isCopied ? 'COPIED TO CLIPBOARD' : 'COPY OUTPUT PAYLOAD' }}</span>
            </button>
          </div>

        </div>
      </div>

      <!-- History Log -->
      <div v-if="history.length > 0" :class="[
        'border rounded-xl p-5 shadow-xl space-y-3 transition-colors',
        isDarkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'
      ]">
        <div :class="['flex items-center justify-between border-b pb-2.5', isDarkMode ? 'border-slate-800' : 'border-slate-200']">
          <h2 :class="['text-xs font-mono font-bold uppercase tracking-widest flex items-center gap-2', isDarkMode ? 'text-slate-400' : 'text-slate-600']">
            <ClipboardList :class="['w-4 h-4', isDarkMode ? 'text-emerald-500' : 'text-emerald-600']" />
            <span>TRANSACTION_LOG</span>
          </h2>
          <button
            @click="history = []"
            :class="['text-[10px] font-mono uppercase tracking-wider transition-colors', isDarkMode ? 'text-slate-600 hover:text-slate-400' : 'text-slate-400 hover:text-slate-600']"
          >
            PURGE_LOGS
          </button>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-3 max-h-48 overflow-y-auto pr-1">
          <div
            v-for="item in history"
            :key="item.id"
            :class="[
              'border rounded-lg p-3 flex flex-col justify-between space-y-1.5 transition-colors',
              isDarkMode
                ? 'bg-slate-950 border-slate-800/80 hover:border-slate-700'
                : 'bg-slate-50 border-slate-200 hover:border-slate-300'
            ]"
          >
            <div class="flex items-center justify-between text-[10px] font-mono">
              <span :class="['font-bold px-1.5 py-0.5 rounded text-[9px] uppercase tracking-wider', item.type === 'encrypt' ? 'bg-emerald-500/10 text-emerald-500 border border-emerald-500/20' : 'bg-indigo-500/10 text-indigo-500 border border-indigo-500/20']">
                {{ item.type === 'encrypt' ? 'ENCRYPT' : 'DECRYPT' }}
              </span>
              <span :class="isDarkMode ? 'text-slate-600' : 'text-slate-400'">{{ item.timestamp }}</span>
            </div>

            <p :class="['text-xs font-mono truncate', isDarkMode ? 'text-slate-300' : 'text-slate-700']">
              {{ item.input }}
            </p>

            <div :class="['flex items-center justify-between text-[10px] font-mono pt-1 border-t', isDarkMode ? 'border-slate-900 text-slate-500' : 'border-slate-200 text-slate-400']">
              <span class="truncate">KEY: {{ item.keyLabel }}</span>
              <button
                @click="inputText = item.output; activeTab = item.type === 'encrypt' ? 'decrypt' : 'encrypt'; showToast('Loaded text from history', 'info')"
                :class="['font-semibold hover:underline shrink-0 ml-2', isDarkMode ? 'text-emerald-500 hover:text-emerald-400' : 'text-emerald-700 hover:text-emerald-800']"
              >
                LOAD_OUTPUT
              </button>
            </div>
          </div>
        </div>
      </div>

    </main>

    <!-- Toast Overlay -->
    <div class="fixed top-5 right-5 z-50 flex flex-col gap-2 pointer-events-none max-w-sm w-full px-4">
      <div
        v-for="toast in toasts"
        :key="toast.id"
        :class="[
          'pointer-events-auto border rounded-lg p-4 flex items-center justify-between gap-3 shadow-2xl backdrop-blur-md',
          isDarkMode
            ? 'bg-emerald-500/10 border-emerald-500/20 text-emerald-200'
            : 'bg-white border-emerald-300 text-emerald-900 shadow-slate-200'
        ]"
      >
        <div class="flex items-center gap-3">
          <div :class="['rounded-full p-1 shrink-0', isDarkMode ? 'bg-emerald-500 text-slate-950' : 'bg-emerald-600 text-white']">
            <CheckCircle2 v-if="toast.type === 'success'" class="w-3.5 h-3.5" />
            <AlertTriangle v-else-if="toast.type === 'error'" class="w-3.5 h-3.5" />
            <Info v-else class="w-3.5 h-3.5" />
          </div>
          <div class="flex flex-col">
            <span :class="['text-[10px] font-bold uppercase tracking-tighter', isDarkMode ? 'text-emerald-400' : 'text-emerald-700']">
              {{ toast.type === 'success' ? 'ACTION SUCCESS' : toast.type === 'error' ? 'PROCESSING ERROR' : 'SYSTEM INFO' }}
            </span>
            <span :class="['text-[11px] font-mono', isDarkMode ? 'text-emerald-200/90' : 'text-slate-700']">{{ toast.message }}</span>
          </div>
        </div>
        <button
          @click="removeToast(toast.id)"
          :class="['transition-colors shrink-0 cursor-pointer', isDarkMode ? 'text-emerald-500/70 hover:text-emerald-300' : 'text-slate-400 hover:text-slate-600']"
        >
          <X class="w-4 h-4" />
        </button>
      </div>
    </div>

    <!-- Footer -->
    <footer :class="['mt-12 max-w-5xl mx-auto w-full flex flex-col sm:flex-row justify-between items-center sm:items-end gap-3 text-[10px] font-mono border-t pt-6 transition-colors', isDarkMode ? 'border-slate-900 text-slate-600' : 'border-slate-200 text-slate-400']">
      <p>&copy; 2026 DMK. All rights reserved.</p>
      <span>CYPHER_VAULT.v1</span>
    </footer>

  </div>
</template>