<template>
  <div class="background"></div>
    <Home />
  <footer>
    <span>© 2025 - {{ currentYear }} 沸腾鱼. All Rights Reserved.</span>
    <a v-if="icpNumber" href="https://beian.miit.gov.cn/" target="_blank" class="beian-link">
      <img src="/beian/icpb.webp" alt="" class="beian-icon" /> {{ icpNumber }}
    </a>
    <a v-if="policenumber" :href="`https://beian.mps.gov.cn/#/query/webSearch?police=${policenumber}`" target="_blank" class="police_link">
      <img src="/beian/gawb.webp" alt="" class="beian-icon" /> {{ policenumber }}
    </a>
  </footer>
</template>

<script setup>
import { ref } from 'vue';
import Home from './components/Home.vue';

const userName = ref(import.meta.env.VITE_APP_USER_NAME);
const icpNumber = ref(import.meta.env.VITE_APP_ICP_NUMBER);
const policenumber = ref(import.meta.env.VITE_APP_POLICE_NUMBER);

const currentYear = new Date().toLocaleDateString('en-US', { timeZone: 'Asia/Shanghai', year: 'numeric' });
</script>