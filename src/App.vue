<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-500 via-purple-200 to-pink-500 text-neutral-800 p-2 sm:p-6">
    <!-- Loading Overlay -->
    <div v-if="isLoading" class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg p-6 text-center">
        <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-purple-500 mx-auto mb-4"></div>
        <p class="text-gray-600">Loading...</p>
      </div>
    </div>

    <!-- Navigation -->
    <!-- <nav class="bg-white/80 backdrop-blur-sm shadow-sm p-4">
      <div class="w-full mx-auto flex justify-between items-center">
        <h1 class="text-2xl font-bold text-gray-800">Gender Reveal</h1>
        <div class="flex gap-2">
          <button 
            @click="currentPage = 'vote'" 
            :class="currentPage === 'vote' ? 'bg-purple-500 text-white' : 'bg-gray-200 text-gray-700'"
            class="px-4 py-2 rounded-lg font-medium transition-colors"
          >
            Vote
          </button>
          <button 
            @click="currentPage = 'tally'" 
            :class="currentPage === 'tally' ? 'bg-purple-500 text-white' : 'bg-gray-200 text-gray-700'"
            class="px-4 py-2 rounded-lg font-medium transition-colors"
          >
            Results
          </button>
          <button 
            @click="currentPage = 'admin'" 
            :class="currentPage === 'admin' ? 'bg-purple-500 text-white' : 'bg-gray-200 text-gray-700'"
            class="px-4 py-2 rounded-lg font-medium transition-colors"
          >
            Admin
          </button>
        </div>
      </div>
    </nav> -->

    <!-- Connection Status -->
    <div v-if="connectionError" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 mx-4 mt-4 rounded">
      <p class="text-sm">⚠️ Connection error: {{ connectionError }}. Some features may not work properly.</p>
    </div>

    <!-- Vote Page -->
    <div v-if="getURLparams('nav') === 'vote'" class="w-full flex items-center justify-center mx-auto text-2xl pt-4 md:pt-28">
      <div class="w-full md:w-6/12 bg-white rounded-2xl shadow-xl p-8 gap-4 grid mt-8 top-42">
        <h2 class="text-4xl sm:text-4xl font-bold text-center mb-8 text-gray-800">🚀 Quick roll call — what team are you on?</h2>
        
        <form @submit.prevent="submitVote" class="grid gap-6 pt-10 text-center text-xl sm:text-2xl">
          <div>
            <label for="name" class="block text-gray-700 mb-2">Your Name <i>or</i>  Nickname</label>
            <input 
              id="name"
              v-model="voterName" 
              type="text" 
              required
              :disabled="isSubmitting"
              class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500 focus:border-transparent outline-none transition-all disabled:opacity-50"
              placeholder="Enter your name"
            >
          </div>

          <div class="grid gap-4 text-center">
            <label class="block text-gray-700 mb-4">Join your team!</label>
            <div class="grid grid-cols-2 gap-4 text-4xl">
              <button 
                type="button"
                @click="selectedGender = 'BOY'"
                :disabled="isSubmitting"
                :class="selectedGender === 'BOY' ? 'bg-blue-500 text-white border-blue-500' : 'bg-white text-blue-500 border-blue-200 hover:bg-blue-50'"
                class="p-6 border-2 rounded-xl font-bold transition-all transform hover:scale-105 disabled:opacity-50 disabled:transform-none h-62 flex flex-col items-center justify-center"
              >
                <div class="text-6xl">
                  👦🏻
                </div>
                <div>
                  BOY
                </div>
              </button>
              <button 
                type="button"
                @click="selectedGender = 'GIRL'"
                :disabled="isSubmitting"
                :class="selectedGender === 'GIRL' ? 'bg-pink-500 text-white border-pink-500' : 'bg-white text-pink-500 border-pink-200 hover:bg-pink-50'"
                class="p-6 border-2 rounded-xl font-bold transition-all transform hover:scale-105 disabled:opacity-50 disabled:transform-none h-62 flex flex-col items-center justify-center"
              >
                <div class="text-6xl">
                  👧🏻
                </div>
                <div>
                  GIRL
                </div>
              </button>
            </div>
          </div>

          <button 
            type="submit" 
            :disabled="!voterName || !selectedGender || isSubmitting"
            class="w-full bg-gradient-to-r from-purple-500 to-pink-500 text-white font-bold py-4 rounded-xl disabled:opacity-50 disabled:cursor-not-allowed hover:from-purple-600 hover:to-pink-600 transition-all transform hover:scale-105 disabled:transform-none"
          >
            {{ isSubmitting ? 'Submitting...' : 'Submit Vote' }}
          </button>
        </form>
      </div>
    </div>

    <!-- Tally Page -->
    <div v-if="getURLparams('nav') === 'tally'" class="w-full mx-auto mt-8 pt-4 md:pt-28">
      <div class="bg-white rounded-2xl shadow-xl p-8 flex gap-10 flex-col">
        <div class="text-center mb-8 flex flex-col gap-2">
          <h2 class="text-3xl font-bold text-gray-800 mb-2">Voting Results</h2>
          <p class="text-gray-600">Total Votes: {{ totalVotes }} • Live Updates: {{ isConnected ? '🟢' : '🔴' }}</p>
          <div v-if="isRevealed" class="flex my-10 p-4 rounded-lg text-center" :class="correctGender === 'BOY' ? 'bg-blue-500' : 'bg-pink-500'">
            <p class="font-bold text-white text-center w-full md:text-9xl sm:text-4xl text-2xl">
              🎉 It's a {{ correctGender == 'BOY' ? '👦🏻' : '👧🏻' }} {{ correctGender }}! 🎉
            </p>
          </div>
        </div>

        <div class="grid md:grid-cols-2 gap-8">
          <!-- Pie Chart -->
          <div class="bg-gray-50 rounded-xl p-6 flex flex-col items-center justify-center gap-10">
            <!-- <h3 class="text-xl font-bold text-center mb-4">Pie Chart</h3> -->
            <div class="relative w-48 h-48 mx-auto">
              <svg width="200" height="200" viewBox="0 0 32 32" class="rounded-full">
                  <!-- Slice 2 (second part, background) -->
                  <circle
                    r="16"
                    cx="16"
                    cy="16"
                    fill="white"
                    :stroke="'oklch(62.3% 0.214 259.815'"
                    stroke-width="32"
                  />
                  
                  <!-- Slice 1 (foreground) -->
                  <circle
                    r="16"
                    cx="16"
                    cy="16"
                    fill="transparent"
                    :stroke="'#f6339a'"
                    stroke-width="32"
                    :stroke-dasharray="`${girlPercentage} ${100 - girlPercentage}`"
                    transform="rotate(-90 16 16)"
                  />
                </svg>
              <!-- <div class="absolute inset-0 flex items-center justify-center">
                <div class="text-center">
                  <div class="text-2xl font-bold text-gray-800">{{ totalVotes }}</div>
                  <div class="text-sm text-gray-600">votes</div>
                </div>
              </div> -->
            </div>
            <!-- <div class="flex justify-center gap-4 mt-4">
              <div class="flex items-center gap-2">
                <div class="w-4 h-4 bg-blue-500 rounded"></div>
                <span class="text-sm">Boy ({{ boyPercentage }}%)</span>
              </div>
              <div class="flex items-center gap-2">
                <div class="w-4 h-4 bg-pink-500 rounded"></div>
                <span class="text-sm">Girl ({{ girlPercentage }}%)</span>
              </div>
            </div> -->
          </div>

          <!-- Bar Chart -->
          <div class="bg-gray-50 rounded-xl p-6 flex w-full items-center ">
            <!-- <h3 class="text-xl font-bold text-center mb-4">Bar Chart</h3> -->
            <div class="space-y-4 w-full gap-8 grid">
              <div class="grid gap-2">
                <div class="flex justify-between items-center mb-2 text-2xl">
                  <span class="font-medium text-blue-700">👦🏻 BOY</span>
                  <span class="font-bold">{{ votes.boy }} ({{ boyPercentage }}%)</span>
                </div>
                <div class="w-full bg-gray-200 rounded-full h-6">
                  <div 
                    class="bg-blue-500 h-6 rounded-full transition-all duration-1000 ease-out"
                    :style="{ width: boyPercentage + '%' }"
                  ></div>
                </div>
              </div>
              <div class="grid gap-2">
                <div class="flex justify-between items-center mb-2 text-2xl">
                  <span class="font-medium text-pink-700">👧🏻 GIRL</span>
                  <span class="font-bold">{{ votes.girl }} ({{ girlPercentage }}%)</span>
                </div>
                <div class="w-full bg-gray-200 rounded-full h-6">
                  <div 
                    class="bg-pink-500 h-6 rounded-full transition-all duration-1000 ease-out"
                    :style="{ width: girlPercentage + '%' }"
                  ></div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Recent Voters -->
        <div class="mt-8">
          <h3 class="text-xl font-bold mb-4">Recent Voters</h3>
          <div v-if="voters.length === 0" class="text-center text-gray-500 py-8">
            No votes yet. Be the first to vote!
          </div>
          <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            <div 
              v-for="voter in voters" 
              :key="voter.id"
              class="bg-gray-50 rounded-lg p-4 flex items-center justify-between"
            >
              <span class="font-medium">{{ voter.voter_name.substr(0,4) }}*****</span>
              <span 
                :class="voter.vote === 'BOY' ? 'text-blue-600 bg-blue-100' : 'text-pink-600 bg-pink-100'"
                class="px-3 py-1 rounded-full text-sm font-bold"
              >
                {{ voter.vote }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Admin Page -->
    <div v-if="getURLparams('nav') === '21232f297a57a5a743894a0e4a801fc3'" class="w-full mx-auto mt-8 pt-4 md:pt-28">
      <div class="bg-white rounded-2xl shadow-xl p-8">
        <div class="space-y-6 grid gap-4 ">
          <div v-if="correctGender === ''" class="text-center grid gap-4 rounded-2xl">
            <label class="block font-medium text-gray-700 mb-4 text-2xl">Set Gender</label>
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 text-5xl">
              <button 
                type="button"
                @click="updateCorrectGender('BOY')"
                :disabled="isUpdatingSettings"
                :class="correctGender === 'BOY' ? 'bg-blue-500 text-white border-blue-500' : 'bg-white text-blue-500 border-blue-200 hover:bg-blue-50'"
                class="p-3 sm:p-4 md:p-6 border-2 rounded-xl font-bold transition-all disabled:opacity-50 h-52 flex items-center justify-center">
                👦🏻 BOY
              </button>
              <button 
                type="button"
                @click="updateCorrectGender('GIRL')"
                :disabled="isUpdatingSettings"
                :class="correctGender === 'GIRL' ? 'bg-pink-500 text-white border-pink-500' : 'bg-white text-pink-500 border-pink-200 hover:bg-pink-50'"
                class="p-3 sm:p-4 md:p-6 border-2 rounded-xl font-bold transition-all disabled:opacity-50 h-52 flex items-center justify-center">
                👧🏻 GIRL
              </button>
            </div>
          </div>

          <div class="grid grid-cols-2 w-full gap-4 text-base sm:text-xl">
            <div class="bg-gray-50 rounded-lg p-4 ">
              <p>Correct Gender: <br> <span class="font-extrabold">{{ correctGender ? 'HAS BEEN SET' : 'NOT SET' }}</span></p>
              <br>
              <p>Revealed: {{ isRevealed ? 'Yes' : 'No' }}</p>
            </div>
            <div class="bg-gray-50 rounded-lg p-4 ">
              <p>Total Votes: <br> <span>{{ voters.length }}</span></p>
              <br>
              <p>Database: {{ isConnected ? 'Connected' : 'Disconnected' }}</p>
            </div>
          </div>

          <button 
            @click="revealGender"
            :disabled="!correctGender || isUpdatingSettings"
            class="w-full bg-gradient-to-r from-yellow-500 to-orange-500 text-white font-bold py-4 rounded-xl disabled:opacity-50 disabled:cursor-not-allowed hover:from-yellow-600 hover:to-orange-600 transition-all transform hover:scale-105 disabled:transform-none"
          >
            {{ isUpdatingSettings ? 'Updating...' : '🎉 Reveal Gender!' }}
          </button>

          <button 
            @click="resetAll"
            :disabled="isUpdatingSettings"
            class="w-full bg-red-500 text-white font-bold py-3 rounded-xl hover:bg-red-600 transition-all disabled:opacity-50"
          >
            {{ isUpdatingSettings ? 'Resetting...' : 'Reset All Data' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { createClient } from '@supabase/supabase-js'

// Supabase client
const supabase = createClient(
  "https://ousmvstnbolbznnkvscg.supabase.co",
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im91c212c3RuYm9sYnpubmt2c2NnIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTA0MDEzOTUsImV4cCI6MjA2NTk3NzM5NX0.F7R4YTx4ofiZdWlvWNihLsuDWG6QMbRPc5k1tTkhIKE"
)

// Reactive data
const currentPage = ref('vote')
const voterName = ref('')
const selectedGender = ref('')
const votes = ref({ boy: 0, girl: 0 })
const voters = ref([])
const correctGender = ref('')
const isRevealed = ref(false)

// Loading and connection states
const isLoading = ref(true)
const isSubmitting = ref(false)
const isUpdatingSettings = ref(false)
const isConnected = ref(true)
const connectionError = ref('')

// Realtime subscription
let votesSubscription = null
let settingsSubscription = null

// Computed properties
const totalVotes = computed(() => votes.value.boy + votes.value.girl)
const boyPercentage = computed(() => 
  totalVotes.value === 0 ? 0 : Math.round((votes.value.boy / totalVotes.value) * 100)
)
const girlPercentage = computed(() => 
  totalVotes.value === 0 ? 0 : Math.round((votes.value.girl / totalVotes.value) * 100)
)

// Methods
const submitVote = async () => {
  if (!voterName.value || !selectedGender.value || isSubmitting.value) return
  
  isSubmitting.value = true
  connectionError.value = ''
  
  try {
    const { error } = await supabase
      .from('votes')
      .insert([
        {
          voter_name: voterName.value,
          vote: selectedGender.value
        }
      ])
    
    if (error) throw error
    loadVotes()
    // Reset form
    voterName.value = ''
    selectedGender.value = ''
    
    // Redirect to tally page
    currentPage.value = 'tally'
    setURLparam('nav', 'tally')
    
  } catch (error) {
    console.error('Error submitting vote:', error)
    connectionError.value = 'Failed to submit vote. Please try again.'
  } finally {
    isSubmitting.value = false
  }
}

const updateCorrectGender = async (gender) => {
  if (isUpdatingSettings.value) return
  
  isUpdatingSettings.value = true
  connectionError.value = ''
  
  try {
    const { error } = await supabase
      .from('admin_settings')
      .update({ setting_value: gender, updated_at: new Date().toISOString() })
      .eq('setting_key', 'correct_gender')
    
    if (error) throw error
    
    correctGender.value = gender
    
  } catch (error) {
    console.error('Error updating correct gender:', error)
    connectionError.value = 'Failed to update setting. Please try again.'
  } finally {
    isUpdatingSettings.value = false
  }
}

const revealGender = async () => {
  if (!correctGender.value || isUpdatingSettings.value) return
  
  isUpdatingSettings.value = true
  connectionError.value = ''
  
  try {
    const { error } = await supabase
      .from('admin_settings')
      .update({ setting_value: 'true', updated_at: new Date().toISOString() })
      .eq('setting_key', 'is_revealed')
    
    if (error) throw error
    
    isRevealed.value = true
    currentPage.value = 'tally'
    
  } catch (error) {
    console.error('Error revealing gender:', error)
    connectionError.value = 'Failed to reveal gender. Please try again.'
  } finally {
    isUpdatingSettings.value = false
  }
}

const resetAll = async () => {
  if (!confirm('Are you sure you want to reset all data? This cannot be undone.') || isUpdatingSettings.value) {
    return
  }
  
  isUpdatingSettings.value = true
  connectionError.value = ''
  
  try {
    // Delete all votes
    const { error: votesError } = await supabase
      .from('votes')
      .delete()
      .neq('id', 0) // Delete all rows
    
    if (votesError) throw votesError
    
    // Reset admin settings
    const { error: settingsError } = await supabase
      .from('admin_settings')
      .update({ setting_value: null, updated_at: new Date().toISOString() })
      .eq('setting_key', 'correct_gender')
    
    if (settingsError) throw settingsError
    
    const { error: revealError } = await supabase
      .from('admin_settings')
      .update({ setting_value: 'false', updated_at: new Date().toISOString() })
      .eq('setting_key', 'is_revealed')
    
    if (revealError) throw revealError
    
    // Reset local state
    votes.value = { boy: 0, girl: 0 }
    voters.value = []
    correctGender.value = ''
    isRevealed.value = false
    voterName.value = ''
    selectedGender.value = ''
    
  } catch (error) {
    console.error('Error resetting data:', error)
    connectionError.value = 'Failed to reset data. Please try again.'
  } finally {
    isUpdatingSettings.value = false
  }
}

const loadVotes = async () => {
  try {
    const { data, error } = await supabase
      .from('votes')
      .select('*')
      .order('created_at', { ascending: false })
    
    if (error) throw error
    
    voters.value = data || []
    isLoading.value = false
    
    // Calculate vote counts
    const boyVotes = data?.filter(vote => vote.vote === 'BOY').length || 0
    const girlVotes = data?.filter(vote => vote.vote === 'GIRL').length || 0
    
    votes.value = { boy: boyVotes, girl: girlVotes }
    
  } catch (error) {
    console.error('Error loading votes:', error)
    connectionError.value = 'Failed to load votes.'
    isConnected.value = false
  }
}

const loadSettings = async () => {
  try {
    const { data, error } = await supabase
      .from('admin_settings')
      .select('*')
    
    if (error) throw error
    
    const settings = {}
    data?.forEach(setting => {
      settings[setting.setting_key] = setting.setting_value
    })
    
    correctGender.value = settings.correct_gender || ''
    isRevealed.value = settings.is_revealed === 'true'
    
  } catch (error) {
    console.error('Error loading settings:', error)
    connectionError.value = 'Failed to load settings.'
    isConnected.value = false
  }
}

const setupRealtimeSubscriptions = () => {
  // Subscribe to votes changes
  votesSubscription = supabase
    .channel('votes_changes')
    .on('postgres_changes', 
      { event: '*', schema: 'public', table: 'votes' }, 
      () => {
        loadVotes()
      }
    )
    .subscribe()
  
  // Subscribe to settings changes
  settingsSubscription = supabase
    .channel('settings_changes')
    .on('postgres_changes', 
      { event: '*', schema: 'public', table: 'admin_settings' }, 
      () => {
        loadSettings()
      }
    )
    .subscribe()

    return [votesSubscription, settingsSubscription]
}

const getURLparams = (param) => {
  const queryString = window.location.search;
  const urlParams = new URLSearchParams(queryString);
  const nav = urlParams.get(param);
  if (nav==undefined) {
    return 'vote'
  }
  return nav
}

// set URL params value
const setURLparam = (param, value) => {
  const url = new URL(window.location);
  url.searchParams.set(param, value);
  window.history.replaceState({}, '', url);
};

// Initialize app
onMounted(async () => {
  try {
    await Promise.all([loadVotes(), loadSettings()])
    setupRealtimeSubscriptions()
    isConnected.value = true
  } catch (error) {
    console.error('Error initializing app:', error)
    isConnected.value = false
  } finally {
    //
  }
})

// Cleanup subscriptions
onUnmounted(() => {
  if (votesSubscription) {
    supabase.removeChannel(votesSubscription)
  }
  if (settingsSubscription) {
    supabase.removeChannel(settingsSubscription)
  }
})
</script>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
