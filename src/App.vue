<script setup>
import { ref, onMounted } from 'vue';
import { GamifiedCaptcha } from 'vue-gamified-captcha';
import { auth, createUserWithEmailAndPassword, isFirebaseConfigured } from './firebase.js';

const email = ref('');
const password = ref('');
const captchaVerified = ref(false);
const loading = ref(false);
const error = ref('');
const gameUrl = ref('');
const siteKey = ref(import.meta.env.VITE_CAPTCHA_SITE_KEY || 'demo_tenant');

const SELECTED_GAMES = [
  "https://conversion.business/sunny-day-maze/",
  "https://conversion.business/smack-that-donkey/",
  "https://conversion.business/danger-point/",
  "https://conversion.business/trend-catcher/",
  "https://conversion.business/turtle-stacker/"
];

onMounted(() => {
  // Randomize the game once on the client
  gameUrl.value = SELECTED_GAMES[Math.floor(Math.random() * SELECTED_GAMES.length)];
});

const handleRegister = async () => {
  if (!captchaVerified.value) {
    error.value = 'Please complete the human verification check first.';
    return;
  }
  
  loading.value = true;
  error.value = '';

  try {
    if (!isFirebaseConfigured()) {
      if (import.meta.env.PROD) {
        console.error("CRITICAL ERROR: Application deployed to production without Firebase credentials. Mock Mode is disabled in production to prevent fake data pollution.");
        error.value = "Site Configuration Error: The database is currently unavailable. Please contact the administrator.";
        loading.value = false;
        return;
      }
      // QA Fallback Mode
      console.warn('Firebase not configured. Mock registration successful.');
      await new Promise(r => setTimeout(r, 800));
      alert('Mock Dashboard Login Success!');
      loading.value = false;
      return;
    }

    await createUserWithEmailAndPassword(auth, email.value, password.value);
    alert('Dashboard Login Success!');
  } catch (err) {
    error.value = err.message || 'Registration failed';
  } finally {
    loading.value = false;
  }
};

const handleHumanVerified = (token) => {
  console.log("Human verified successfully! Secure payload:", token);
  captchaVerified.value = true;
  error.value = '';
};
</script>

<template>
  <div class="min-h-screen flex items-center justify-center p-5 pt-16 pb-24 bg-background text-foreground">
    <div class="w-full max-w-md bg-surface border border-border p-8 rounded-2xl shadow-2xl relative overflow-hidden">
      <!-- Decorative Top Line -->
      <div class="absolute top-0 left-0 right-0 h-1 bg-gradient-to-r from-primary to-emerald-500"></div>
      
      <h2 class="text-3xl font-bold mb-2">Create Account</h2>
      <p class="text-muted mb-8 text-sm">Start your free trial today. (Vue Edition)</p>

      <form @submit.prevent="handleRegister" class="flex flex-col gap-5">
        <div v-if="error" class="bg-red-500/10 text-red-500 p-3 rounded-lg text-sm border border-red-500/20">{{ error }}</div>
        
        <div>
          <label class="block text-sm font-medium mb-1.5 text-foreground/80">Work Email</label>
          <input 
            type="email" 
            required
            v-model="email"
            class="w-full bg-background border border-border rounded-lg px-4 py-3 focus:outline-none focus:border-primary transition-colors text-sm"
            placeholder="name@company.com"
          />
        </div>

        <div>
          <label class="block text-sm font-medium mb-1.5 text-foreground/80">Password</label>
          <input 
            type="password" 
            required
            v-model="password"
            class="w-full bg-background border border-border rounded-lg px-4 py-3 focus:outline-none focus:border-primary transition-colors text-sm"
            placeholder="••••••••"
          />
        </div>

        <div class="mt-2">
          <p class="text-sm font-medium mb-2.5 text-foreground/80">Verification</p>
          <template v-if="gameUrl">
            <GamifiedCaptcha 
              :siteKey="siteKey"
              :gameUrl="gameUrl"
              @humanVerified="handleHumanVerified"
            />
          </template>
          <div v-else class="h-[400px] w-full bg-surface border border-border rounded-xl flex items-center justify-center animate-pulse text-muted">
            Selecting Security Validation...
          </div>
        </div>

        <button 
          type="submit" 
          :disabled="!captchaVerified || loading"
          class="w-full bg-primary text-primary-foreground py-3.5 rounded-lg font-bold hover:brightness-110 transition-all disabled:opacity-50 disabled:cursor-not-allowed mt-2 shadow-[0_0_15px_rgba(59,130,246,0.2)] disabled:shadow-none"
        >
          {{ loading ? 'Creating Account...' : 'Complete Setup' }}
        </button>

        <p class="text-center text-sm text-muted mt-4">
          Already have an account? <a href="#" class="text-primary hover:underline">Log in</a>
        </p>
      </form>
    </div>
  </div>
</template>
