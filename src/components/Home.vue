<template>
  <div class="content">
    <div class="user-profile-container">
      <div class="user-profile-image" v-motion-pop>
        <img :src="profileImage" alt="头像" @click.stop="toggleInfo">
        <span class="status-ball"></span>
      </div>
      <div class="user-name" v-motion-slide-left>
        <h1>Hi,</h1>
        <h1>I'm <span class="name-style">{{ userName }}</span></h1>
      </div>
    </div>
    <div class="description">
      <p ref="descriptionElement"></p>
    </div>
    <div class="contact-section" v-motion-pop>
      <template v-for="contact in contacts" :key="contact.type">
        <a v-if="contact.url && contact.type !== 'Email'" :href="contact.url" target="_blank" class="contact-item" :style="{ '--hover-color': contact.hoverColor }">
          <i :class="contact.icon"></i>
          <span class="tooltip">{{ contact.type }}</span>
        </a>
        <span v-else @click="handleContactClick(contact)" class="contact-item" :style="{ '--hover-color': contact.hoverColor }">
          <i :class="contact.icon"></i>
          <span class="tooltip">{{ contact.type }}</span>
        </span>
      </template>
      <span class="contact-item" @click="toggleDarkMode" :style="{ '--hover-color': isDarkMode ? '#ffcc00' : '#666' }">
        <i :class="darkModeIconClass"></i>
        <span class="tooltip">{{ isDarkMode ? '浅色' : '深色' }}</span>
      </span>
    </div>
    <Website /> 
    <VisitTimer />

    <Transition name="fade">
      <div v-if="showAbout" class="overlay" @click="showAbout = false">
        <div class="modal-content">
          <AboutPage @close="showAbout = false" />
        </div>
      </div>
    </Transition>

    <Transition name="fade">
      <div v-if="showQR" class="overlay" @click="hideQRCode">
        <div class="modal-content">
          <img :src="qrCodeSrc" alt="QR Code" class="qr-image" @click.stop>
        </div>
      </div>
    </Transition>

    <Transition name="fade">
      <div v-if="showEmailModal" class="overlay" @click="showEmailModal = false">
        <div class="modal-content email-modal" @click.stop v-motion-pop>
          <div class="email-modal-header">
            <h3>选择联系方式 / Choose Contact</h3>
          </div>
          <div class="email-options">
            <a v-for="email in emailOptions" :key="email.address" :href="`mailto:${email.address}`" class="email-item" @click="showEmailModal = false">
              <div class="email-item-icon en">{{ email.iconLabel }}</div>
              <div class="email-item-info">
                <span class="lang">{{ email.lang }}</span>
                <span class="address">{{ email.address }}</span>
              </div>
              <i class="fas fa-chevron-right"></i>
            </a>
          </div>
          <button class="close-modal-btn" @click="showEmailModal = false">
            <i class="fas fa-times"></i>
          </button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import contactsData from '../config/links.json';
import Website from './Website.vue';
import AboutPage from './AboutPage.vue';
import VisitTimer from './VisitTimer.vue';
import Typed from 'typed.js';

const contacts = ref(contactsData);
const showQR = ref(false);
const showAbout = ref(false);
const showEmailModal = ref(false);
const qrCodeSrc = ref('');
const profileImage = ref(import.meta.env.VITE_APP_PROFILE_IMAGE_URL);
const userName = ref(import.meta.env.VITE_APP_USER_NAME);
const descriptionElement = ref(null);
const emailOptions = ref(contactsData.find(c => c.type === 'Email')?.emails || []);

let typedInstance = null;

const fetchHitokoto = async () => {
  const quotes = [];
  try {
    // 并发请求 5 条一言
    const requests = Array(5).fill().map(() => fetch('https://v1.hitokoto.cn').then(res => res.json()));
    const results = await Promise.all(requests);
    return results.map(r => r.hitokoto);
  } catch (error) {
    console.error('获取一言失败', error);
    return ["你好鸭，欢迎来到我的主页！！"];
  }
};

const initializeTyped = (strings) => {
  typedInstance = new Typed(descriptionElement.value, {
    strings: strings,
    typeSpeed: 100,
    backSpeed: 50,
    showCursor: true,
    cursorChar: '|',
    loop: true,
    backDelay: 2000,
  });
};

onMounted(async () => {
  const strings = await fetchHitokoto();
  initializeTyped(strings);
});

const toggleQRCode = (qrCode) => {
  qrCodeSrc.value = qrCode || '';
  showQR.value = !showQR.value;
};

const hideQRCode = () => {
  showQR.value = false;
};

const handleContactClick = (contact) => {
  if (contact.type && contact.type.toLowerCase() === 'email') {
    showEmailModal.value = true;
  } else if (contact.qrCode) {
    toggleQRCode(contact.qrCode);
  }
};

const toggleInfo = () => {
  showAbout.value = !showAbout.value;
};

const isDarkMode = ref(false);
const darkModeIconClass = ref('fas fa-moon');

const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value;
  document.body.classList.toggle('dark-mode', isDarkMode.value);

  localStorage.setItem('darkMode', isDarkMode.value);

  darkModeIconClass.value = isDarkMode.value ? 'fas fa-sun' : 'fas fa-moon';
};

onMounted(() => {
  const savedDarkMode = localStorage.getItem('darkMode');
  if (savedDarkMode !== null) {
    isDarkMode.value = savedDarkMode === 'true';
    document.body.classList.toggle('dark-mode', isDarkMode.value);
  }

  darkModeIconClass.value = isDarkMode.value ? 'fas fa-sun' : 'fas fa-moon';
});
</script>

<style scoped>
.content {
  flex: 1;
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
  gap: 30px;
  margin-top: 20px;
  max-width: 100%;
  overflow-x: hidden;
  padding: 0 15px;

  .user-profile-container {
    display: flex;
    align-items: center;
    gap: 30px;
  }

  .user-profile-image {
    display: flex;
    border-radius: 50%;
    box-shadow: 0 2px 8px var(--shadow-color);
    padding: 5px;
    border: 3px solid var(--border-color);
    position: relative;

    img {
      width: 150px;
      height: 150px;
      border-radius: 50%;
      background-size: cover;
      background-position: center;
    }

    .status-ball {
      position: absolute;
      background: #00c800;
      width: 2em;
      height: 2em;
      border-radius: 20px;
      border: 3px solid #eee;
      bottom: 5px;
      right: 15px;
      display: flex;
      justify-content: center;
      align-items: center;
      transition: all 0.3s ease;
      z-index: 1;
      cursor: pointer;
      overflow: hidden;

      &::before {
        content: "在线中";
        color: #00c800;
        opacity: 0;
        transition: opacity 0.3s ease-in-out, color 0.1s ease-in-out;
      }

      &:hover {
        width: 4.5em;
        height: 2em;
      }

      &:hover::before {
        opacity: 1;
        color: #eee;
      }
    }
  }

  .user-name {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    font-size: 1.3em;

    h1 {
      margin: 0;
    }
  }

  .name-style {
    position: relative;

    &:before {
      position: absolute;
      border-radius: 5px;
      bottom: 0;
      left: 50%;
      transform: translate(-50%);
      z-index: -1;
      content: "";
      background: #ffcc00ad;
      height: 30%;
      width: 110%;
      transition: height 0.3s ease-in-out;
    }
    &:hover::before {
      height: 60%;
    }
  }

  .description {
    display: flex;
    min-height: 32px;
    width: 100%;
    max-width: 500px;
    font-family: 'Georgia', serif;
    font-size: 1.2rem;
    white-space: nowrap;
    text-overflow: ellipsis;
    align-items: center;
    justify-content: center;
    transition: all 0.3s ease-in-out;
    padding: 0 20px;
    box-sizing: border-box;

    &::before,
    &::after {
      content: '"';
      font-size: 1.5em;
      color: #999;
      margin: 0 10px;
    }

    p {
      margin: 0;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  }

  .contact-section {
    display: flex;
    justify-content: center;
    gap: 20px;
    padding: 5px 10px;
    border: 1px solid transparent;
    border-radius: var(--border-radius);
    transition: all 0.3s ease-in-out;
    flex-wrap: wrap;
    max-width: 100%;

    .contact-item {
      color: var(--text-color);
      font-size: var(--icon-size);
      cursor: pointer;
      transition: transform 0.3s ease-in-out, color 0.3s ease-in-out;
      position: relative;

      .fas.fa-moon {
        width: 20px;
        height: 20px;
        display: inline-flex;
        justify-content: center;
        align-items: center;
      }

      &:hover {
        transform: translateY(-5px) rotate(10deg);
        color: var(--hover-color);

        .tooltip {
          opacity: 1;
          transform: translate(-50%, 0);
        }
      }

      .tooltip {
        position: absolute;
        bottom: 100%;
        left: 50%;
        transform: translate(-50%, 10px);
        opacity: 0;
        transition: opacity 0.3s ease, transform 0.3s ease;
        white-space: nowrap;
        pointer-events: none;
      }
    }

    &:hover {
      backdrop-filter: blur(10px);
      border: 1px solid var(--border-color);
      box-shadow: 0 2px 8px var(--shadow-color);
      background-color: rgba(var(--background-color-rgb), 0.2);
    }
  }

  .overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.6);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .fade-enter-active,
  .fade-leave-active {
    transition: all 0.3s ease-out;
    
    .modal-content {
      transition: all 0.3s ease-out;
    }
  }

  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
    
    .modal-content {
      transform: translateY(30px) scale(0.8);
      opacity: 0;
    }
  }

  .fade-enter-to,
  .fade-leave-from {
    opacity: 1;
    
    .modal-content {
      transform: translateY(0) scale(1);
      opacity: 1;
    }
  }

  .qr-image {
    width: 300px;
    height: 300px;
    background: white;
    padding: 20px;
    border-radius: var(--border-radius);
    box-shadow: 0 4px 8px var(--shadow-color);
    transition: all 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
    &:hover {
      transform: scale(1.03) translateY(-5px);
      box-shadow: 0 15px 30px -10px rgba(0, 0, 0, 0.2);
    }
  }

  /* Email Modal Styles */
  .email-modal {
    background: rgba(var(--background-color-rgb), 0.95) !important;
    backdrop-filter: blur(20px);
    width: 380px;
    padding: 30px !important;
    border-radius: 20px !important;
    border: 1px solid var(--border-color);
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
    position: relative;
    display: flex;
    flex-direction: column;
    gap: 20px;

    .email-modal-header h3 {
      margin: 0;
      font-size: 1.1rem;
      color: var(--text-color);
      opacity: 0.9;
      text-align: center;
    }

    .email-options {
      display: flex;
      flex-direction: column;
      gap: 12px;
    }

    .email-item {
      display: flex;
      align-items: center;
      gap: 15px;
      padding: 12px 16px;
      border-radius: 14px;
      background: rgba(var(--background-color-rgb), 0.3);
      border: 1px solid var(--border-color);
      transition: all 0.3s ease;
      text-decoration: none;
      color: var(--text-color);

      .email-item-icon {
        width: 40px;
        height: 40px;
        display: flex;
        align-items: center;
        justify-content: center;
        border-radius: 10px;
        font-weight: bold;
        font-size: 0.8rem;
        flex-shrink: 0;
        
        &.en { background: #4285f433; color: #4285f4; }
        &.cn { background: #ff4b4b33; color: #ff4b4b; }
      }

      .email-item-info {
        flex: 1;
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        
        .lang {
          font-size: 0.9rem;
          font-weight: 600;
        }
        
        .address {
          font-size: 0.75rem;
          opacity: 0.6;
        }
      }

      &:hover {
        transform: translateY(-2px);
        border-color: var(--hover-link-color);
        background: rgba(var(--background-color-rgb), 0.1);
      }
    }

    .close-modal-btn {
      position: absolute;
      top: 15px;
      right: 15px;
      background: none;
      border: none;
      color: var(--text-color);
      opacity: 0.5;
      cursor: pointer;
      font-size: 1.1rem;
      transition: all 0.3s ease;
      
      &:hover {
        opacity: 1;
        transform: rotate(90deg);
      }
    }
  }
}

@media screen and (max-width: 768px) {
  .content {
    gap: 15px;
  }
  .content .user-profile-container {
    flex-direction: column;
    gap: 0;
  }

  .email-modal {
    width: 90%;
    padding: 20px !important;
  }

  h1 {
    font-size: 1.5em;
  }
}
</style>