<script setup lang="ts">
import { h, ref, onMounted } from 'vue';
import { NDropdown, type DropdownOption, NModal, NInput, NInputNumber, NButton, NGrid, NGridItem, useMessage, NImage, NForm, NFormItem, NSwitch, NTag, NSelect, NConfigProvider, NSpin, NP, NA, lightTheme, darkTheme } from 'naive-ui';
import settingSvgUrl from '@/assets/img/setting.svg?url';
import { usePromptStore } from '@/stores/modules/prompt';
import { storeToRefs } from 'pinia';
import ChatNavItem from './ChatNavItem.vue';
import type { Component } from 'vue';
import { isMobile } from '@/utils/utils';
import CreateImage from '@/components/CreateImage/CreateImage.vue';
import { useChatStore } from '@/stores/modules/chat';
import { useUserStore } from '@/stores/modules/user';

const isShowMore = ref(false);
const isShowSettingModal = ref(false);
const isShowAdvancedSettingModal = ref(false);
const isShowSetAboutModal = ref(false);
const isShowCookieModal = ref(false);
const isShowLoginModal = ref(false);
const isShowIframe = ref(false);
const userToken = ref('');
const userKievRPSSecAuth = ref('');
const userMUID = ref('');
const userRwBf = ref('');
const message = useMessage();
const promptStore = usePromptStore();
const { isShowPromptSotre } = storeToRefs(promptStore);
const isShowClearCacheModal = ref(false);
const isShowCreateImageModal = ref(false);
const chatStore = useChatStore();
const { isShowChatServiceSelectModal } = storeToRefs(chatStore);
const userStore = useUserStore();
const localVersion = __APP_INFO__.version;
const lastVersion = ref('正在加載...');
const { historyEnable, themeMode, uiVersion, fullCookiesEnable, cookiesStr, enterpriseEnable, customChatNum, gpt4tEnable, sydneyEnable, sydneyPrompt, passServer } = storeToRefs(userStore)
let cookiesEnable = ref(false);
let cookies = ref('');
let history = ref(true);
let themeModeSetting = ref('auto');
let uiVersionSetting = ref('v3');
let theme = ref(lightTheme);
let settingIconStyle = ref({
  filter: 'invert(70%)',
})
let passingCFChallenge = ref(false);
const enterpriseSetting = ref(false);
const customChatNumSetting = ref(0);
const gpt4tSetting = ref(true);
const sydneySetting = ref(false);
const sydneyPromptSetting = ref('');
const passServerSetting = ref('');
const author = ref('');
const getCookieTip = ref('正在獲取 Cookie , 稍等...');

const GetLastVersion = async () => {
  const res = await fetch('https://api.github.com/repos/Harry-zklcdc/go-proxy-bingai/releases/latest');
  const json = await res.json();
  lastVersion.value = json.tag_name;
};

const navType = {
  login: 'login',
  github: 'github',
  chatService: 'chatService',
  promptStore: 'promptStore',
  setting: 'setting',
  compose: 'compose',
  createImage: 'createImage',
  advancedSetting: 'advancedSetting',
  reset: 'reset',
  about: 'about',
};
const navConfigs = ref([
  {
    key: navType.setting,
    label: '設定',
  },
  {
    key: navType.compose,
    label: '撰寫文章',
    url: '/web/compose.html',
  },
  {
    key: navType.createImage,
    label: '映像建立',
  },
  {
    key: navType.reset,
    label: '一鍵重置',
  },
  {
    label: '淺色',
    value: 'light',
  }, {
    label: '深色',
    value: 'dark',
  }, {
    label: '跟隨作業系統',
    value: 'auto',
  }
]);

const uiVersionOptions = ref([
  {
    label: 'V1',
    value: 'v1',
  },
  {
    label: 'V2',
    value: 'v2',
  },
  {
    label: 'V3',
    value: 'v3',
  }
]);

const themeModeOptions = ref([
  {
    label: '淺色',
    value: 'light',
  }, {
    label: '深色',
    value: 'dark',
  }, {
    label: '跟隨作業系統',
    value: 'auto',
  }
]);

onMounted(() => {
  if (themeMode.value == 'light') {
    theme.value = lightTheme;
    settingIconStyle.value = { filter: 'invert(0%)' }
  } else if (themeMode.value == 'dark') {
    theme.value = darkTheme;
    settingIconStyle.value = { filter: 'invert(70%)' }
  } else if (themeMode.value == 'auto') {
    if (window.matchMedia("(prefers-color-scheme: dark)").matches) {
      theme.value = darkTheme;
      settingIconStyle.value = { filter: 'invert(70%)' }
    } else {
      theme.value = lightTheme;
      settingIconStyle.value = { filter: 'invert(0%)' }
    }
  }
})

const sleep = async (ms: number) => {
  return new Promise(resolve => setTimeout(resolve, ms));
}

const renderDropdownLabel = (option: DropdownOption) => {
  return h(ChatNavItem as Component, { navConfig: option });
};

const handleSelect = async (key: string) => {
  switch (key) {
    case navType.setting:
      {
        isShowSettingModal.value = true;
      }
      break;
    case navType.createImage:
      {
        if (!userStore.sysConfig?.isSysCK && !userStore.getUserToken()) {
          message.warning('體驗畫圖功能需先登入');
        }
        isShowCreateImageModal.value = true;
      }
      break;
    case navType.reset:
      {
        isShowClearCacheModal.value = true;
      }
      break;
    case navType.about:
      {
        const S = base58Decode(_G.S);
        let tmpA = [];
        for (let i = 0; i < _G.SP.length; i++) {
          tmpA.push(S[_G.SP[i]]);
        }
        author.value = base58Decode(tmpA.join(''));
        isShowSetAboutModal.value = true;
        GetLastVersion();
      }
      break;
    default:
      break;
  }
};

const settingMenu = (key: string) => {
  switch(key) {
    case 'autoPassCFChallenge':
      {
      autoPassCFChallenge()
      }
      break;
    case 'login':
      {
        isShowLoginModal.value = true;
        isShowIframe.value = false;
      }
      break;
    case 'chatService':
      {
        isShowChatServiceSelectModal.value = true;
        chatStore.checkAllSydneyConfig();
      }
      break;
    case 'cookieSetting':
      {
        userToken.value = userStore.getUserToken();
        userKievRPSSecAuth.value = userStore.getUserKievRPSSecAuth();
        userMUID.value = userStore.getUserMUID();
        userRwBf.value = userStore.getUserRwBf();
        history.value = historyEnable.value;
        cookiesEnable.value = fullCookiesEnable.value;
        if (cookiesEnable.value) { cookies.value = cookiesStr.value; }
        isShowCookieModal.value = true;
      }
      break;
    case 'promptStore':
      {
        isShowPromptSotre.value = true;
      }
      break;
    case 'advancedSetting':
      {
        history.value = historyEnable.value;
        themeModeSetting.value = themeMode.value;
        uiVersionSetting.value = uiVersion.value;
        enterpriseSetting.value = enterpriseEnable.value;
        customChatNumSetting.value = customChatNum.value;
        gpt4tSetting.value = gpt4tEnable.value;
        sydneySetting.value = sydneyEnable.value;
        sydneyPromptSetting.value = sydneyPrompt.value;
        passServerSetting.value = passServer.value;
        isShowAdvancedSettingModal.value = true;
      }
      break;
    default:
      return;
  }
}

const resetCache = async () => {
  isShowClearCacheModal.value = false;
  await userStore.resetCache();
  message.success('清理完成');
  window.location.href = '/';
};

const saveSetting = () => {
  if (cookiesEnable.value) {
    userStore.saveCookies(cookies.value);
    cookiesStr.value = cookies.value;
  } else {
    if (!userToken.value) {
      message.warning('請先填入用戶 _U Cookie');
    } else {
      userStore.saveUserToken(userToken.value);
    }
    if (!userKievRPSSecAuth.value) {
      message.warning('請先填入用戶 KievRPSSecAuth Cookie');
    } else {
      userStore.saveUserKievRPSSecAuth(userKievRPSSecAuth.value);
    }
    if (!userRwBf.value) {
      message.warning('請先填入用戶 _RwBf Cookie');
    } else {
      userStore.saveUserRwBf(userRwBf.value);
    }
    if (!userMUID.value) {
      message.warning('請先填入用戶 MUID Cookie');
    } else {
      userStore.saveUserMUID(userMUID.value);
    }
  }
  fullCookiesEnable.value = cookiesEnable.value;
  isShowSettingModal.value = false;
};

const saveAdvancedSetting = () => {
  historyEnable.value = history.value;
  const tmpEnterpris = enterpriseEnable.value;
  enterpriseEnable.value = enterpriseSetting.value;
  customChatNum.value = customChatNumSetting.value;
  const tmpGpt4t = gpt4tEnable.value, tmpSydney = sydneyEnable.value, tmpuiVersion = uiVersion.value;
  gpt4tEnable.value = gpt4tSetting.value;
  sydneyEnable.value = sydneySetting.value;
  sydneyPrompt.value = sydneyPromptSetting.value;
  uiVersion.value = uiVersionSetting.value;
  userStore.setPassServer(passServerSetting.value)

  const serpEle = document.querySelector('cib-serp');
  const sidepanel = serpEle?.shadowRoot?.querySelector('cib-conversation')?.querySelector('cib-side-panel')?.shadowRoot?.querySelector('.main')
  const threadsHeader = sidepanel?.querySelector('.threads-header') as HTMLElement;
  const threadsContainer = sidepanel?.querySelector('.threads-container') as HTMLElement;
  if (!isMobile()) {
    if (history.value && userStore.getUserToken() && !enterpriseEnable.value) {
      if (tmpuiVersion === 'v2') {
        threadsHeader.style.display = 'flex'
        threadsContainer.style.display = 'block'
      } else {
        CIB.vm.sidePanel.panels = [
          { type: 'threads', label: '近期活動' },
          { type: 'plugins', label: '插件' }
        ]
      }
    } else {
      if (tmpuiVersion === 'v2') {
        threadsHeader.style.display = 'none'
        threadsContainer.style.display = 'none'
      } else {
        CIB.vm.sidePanel.panels = [
          { type: 'plugins', label: '插件' }
        ]
        CIB.vm.sidePanel.selectedPanel = 'plugins'
      }
    }
  }

  themeMode.value = themeModeSetting.value;
  if (themeModeSetting.value == 'light') {
    CIB.changeColorScheme(0);
    theme.value = lightTheme;
    settingIconStyle.value = { filter: 'invert(0%)' }
  } else if (themeModeSetting.value == 'dark') {
    CIB.changeColorScheme(1);
    theme.value = darkTheme;
    settingIconStyle.value = { filter: 'invert(70%)' }
  } else if (themeModeSetting.value == 'auto') {
    if (window.matchMedia("(prefers-color-scheme: dark)").matches) {
      CIB.changeColorScheme(1);
      theme.value = darkTheme;
      settingIconStyle.value = { filter: 'invert(70%)' }
    } else {
      CIB.changeColorScheme(0);
      theme.value = lightTheme;
      settingIconStyle.value = { filter: 'invert(0%)' }
    }
  }
  isShowAdvancedSettingModal.value = false;
  if (tmpEnterpris != enterpriseSetting.value || tmpSydney != sydneySetting.value || tmpGpt4t != gpt4tSetting.value || tmpuiVersion != uiVersionSetting.value) {
    window.location.href = '/';
  }
}

const newWindow = () => {
  window.open("/fd/auth/signin?action=interactive&provider=windows_live_id&return_url=https%3a%2f%2fwww.bing.com%2fchat%3fq%3dBing%2bAI%26FORM%3dhpcodx%26wlsso%3d1%26wlexpsignin%3d1&src=EXPLICIT&sig=001DD71D5A386F753B1FC3055B306E8F", "_blank");
}

const loginHandel = async ()=> {
  isShowIframe.value = true;
  getCookieTip.value = '正在獲取 Cookie , 稍等...';
  window.addEventListener('message', function (e) {
    const d = e.data
    if (d.cookies != "" && d.cookies != null && d.cookies != undefined) {
      userStore.saveCookies(d.cookies);
      cookiesStr.value = d.cookies;
      message.success('已成功登入');
      isShowLoginModal.value = false;
      window.location.href = '/';
    }
  })
  await sleep(1500);
  getCookieTimeoutHandel();
  const iframe = document.getElementById('login');
  const S = base58Decode(_G.S);
  let tmpA = [];
  for (let i = 0; i < _G.SP.length; i++) {
    tmpA.push(S[_G.SP[i]]);
  }
  const e = base58Decode(tmpA.join(''));
  (iframe as any).contentWindow.postMessage({
    IG: _G.IG,
    T: await aesEncrypt(e, _G.IG),
  }, '*');
}

const getCookieTimeoutHandel = async() => {
  await sleep(3000)
  getCookieTip.value = '獲取 Cookie 時間太久, 請檢查油猴插件和脚本是否安裝正確';
}

const autoPassCFChallenge = async () => {
  passingCFChallenge.value = true;
  const S = base58Decode(_G.S);
  let tmpA = [];
  for (let i = 0; i < _G.SP.length; i++) {
    tmpA.push(S[_G.SP[i]]);
  }
  const e = base58Decode(tmpA.join(''));
  let resq = await fetch('/pass', {
    credentials: 'include',
    method: "POST", // *GET, POST, PUT, DELETE, etc.
    mode: "cors", // no-cors, *cors, same-origin
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      "IG": _G.IG,
      "T": await aesEncrypt(e, _G.IG),
    }),
  }).then((res) => res.json())
  .catch(() => {
    message.error('人機驗證失敗, 請重試');
    passingCFChallenge.value = false;
  })
  if (resq['result'] != null && resq['result'] != undefined) {
    userStore.saveCookies(resq['result']['cookies']);
    cookiesStr.value = resq['result']['cookies'];
    message.success('自動通過人機驗證成功');
    passingCFChallenge.value = false;
    window.location.href = '/';
  } else {
    message.error('人機驗證失敗, 請重試');
    passingCFChallenge.value = false;
  }
}
</script>

<template>
  <NConfigProvider :theme="theme">
    <NDropdown v-if="isMobile()" class="select-none" :show="isShowMore" :options="navConfigs" :render-label="renderDropdownLabel" @select="handleSelect">
      <NImage class="fixed top-6 right-4 cursor-pointer z-50" :src="settingSvgUrl" alt="設定選單" :preview-disabled="true" @click="isShowMore = !isShowMore" :style="settingIconStyle"></NImage>
    </NDropdown>
    <NDropdown v-else class="select-none" trigger="hover" :options="navConfigs" :render-label="renderDropdownLabel" @select="handleSelect">
      <NImage class="fixed top-6 right-6 cursor-pointer z-50" :src="settingSvgUrl" alt="設定選單" :preview-disabled="true" :style="settingIconStyle"></NImage>
    </NDropdown>
    <NModal v-model:show="isShowLoginModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-3xl py-2">登入MS賬戶</div>
      </template>
      <div v-if="!isShowIframe" style="margin-top:12px; margin-bottom:24px">
        <NP>
          使用此功能之前，請安裝<NA href="https://www.tampermonkey.net/">油猴插件</NA>, 并安裝<NA href="https://gist.github.com/Harry-zklcdc/2bfed48b5690efb0891263df85ce2537/raw/go-proxy-bingai.user.js">此脚本</NA>
          <br>
          請點擊下方「打開登陸頁面」按鈕, 在新的頁面登入成功後點擊確認
        </NP>
      </div>
      <div v-else>
        <NSpin size="large" :description="getCookieTip" style="margin: 0 auto; width: 100%" />
        <iframe id="login" src="https://www.bing.com/" style="border: none; width: 0; height: 0" />
      </div>
      <template #action>
        <NButton size="large" type="info" @click="newWindow">打開登陸頁面</NButton>
        <NButton size="large" @click="isShowLoginModal = false">Cancel</NButton>
        <NButton ghost size="large" type="info" @click="loginHandel">確認</NButton>
      </template>
    </NModal>
    <NModal v-model:show="isShowSettingModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-3xl py-2">設定</div>
      </template>
      <NForm ref="formRef" label-placement="left" label-width="auto" require-mark-placement="right-hanging" style="margin-top: 16px;">
        <NGrid x-gap="0" :cols="2">
          <NGridItem>
            <NFormItem path="cookiesEnable" label="自動人機驗證">
              <NButton type="info" :loading="passingCFChallenge" @click="settingMenu('autoPassCFChallenge')">啓動</NButton>
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem path="cookiesEnable" label="登入MS賬戶">
              <NButton type="info" @click="settingMenu('login')">開啓</NButton>
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem path="cookiesEnable" label="服務選擇">
              <NButton type="info" @click="settingMenu('chatService')">開啓</NButton>
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem path="cookiesEnable" label="Cookie 設定">
              <NButton type="info" @click="settingMenu('cookieSetting')">開啓</NButton>
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem path="cookiesEnable" label="提示詞庫">
              <NButton type="info" @click="settingMenu('promptStore')">開啓</NButton>
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem path="cookiesEnable" label="進階設定">
              <NButton type="info" @click="settingMenu('advancedSetting')">開啓</NButton>
            </NFormItem>
          </NGridItem>
        </NGrid>
      </NForm>
      <template #action>
        <NButton ghost size="large" type="info" @click="isShowSettingModal = false">確認</NButton>
      </template>
    </NModal>
    <NModal v-model:show="isShowCookieModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-3xl py-2">Cookie 設定</div>
      </template>
      <NForm ref="formRef" label-placement="left" label-width="auto" require-mark-placement="right-hanging" style="margin-top: 16px;">
        <NFormItem path="cookiesEnable" label="完整 Cookie">
          <NSwitch v-model:value="cookiesEnable" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="token" label="Token">
          <NInput size="large" v-model:value="userToken" type="text" placeholder="用戶 Cookie ,僅需 _U 的值" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="token" label="KievRPSSecAuth">
          <NInput size="large" v-model:value="userKievRPSSecAuth" type="text" placeholder="用戶 Cookie ,僅需 KievRPSSecAuth 的值" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="token" label="_RwBf">
          <NInput size="large" v-model:value="userRwBf" type="text" placeholder="用戶 Cookie ,僅需 _RwBf 的值" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="token" label="MUID">
          <NInput size="large" v-model:value="userMUID" type="text" placeholder="用戶 Cookie ,僅需 MUID 的值" />
        </NFormItem>
        <NFormItem v-show="cookiesEnable" path="token" label="Cookies">
          <NInput size="large" v-model:value="cookies" type="text" placeholder="完整用戶 Cookie" />
        </NFormItem>
      </NForm>
      <template #action>
        <NButton size="large" @click="isShowSettingModal = false">Cancel</NButton>
        <NButton ghost size="large" type="info" @click="saveSetting">Save</NButton>
      </template>
    </NModal>
    <NModal v-model:show="isShowAdvancedSettingModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-3xl py-2">進階設定</div>
      </template>
      <NForm ref="formRef" label-placement="left" label-width="auto" require-mark-placement="right-hanging"
        style="margin-top: 16px;">
        <NGrid x-gap="0" :cols="2">
          <NGridItem>
            <NFormItem path="history" label="歷史記錄">
              <NSwitch v-model:value="history" />
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem path="enterpriseEnable" label="企業版">
              <NSwitch v-model:value="enterpriseSetting" />
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem path="gpt4tEnable" label="GPT4 Turbo">
              <NSwitch v-model:value="gpt4tSetting" />
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem path="sydneyEnable" label="增强模式">
              <NSwitch v-model:value="sydneySetting" />
            </NFormItem>
          </NGridItem>
        </NGrid>
        <NFormItem path="sydneyPrompt" label="人機驗證伺服器">
          <NInput size="large" v-model:value="passServerSetting" type="text" placeholder="人機驗證伺服器" />
        </NFormItem>
        <NFormItem path="sydneyPrompt" label="提示詞">
          <NInput size="large" v-model:value="sydneyPromptSetting" type="text" placeholder="增强模式提示詞" />
        </NFormItem>
        <NFormItem path="uiVersion" label="UI 版本">
          <NSelect v-model:value="uiVersionSetting" :options="uiVersionOptions" size="large" placeholder="請選擇 UI 版本" />
        </NFormItem>
        <NFormItem path="themeMode" label="主題樣式">
          <NSelect v-model:value="themeModeSetting" :options="themeModeOptions" size="large" placeholder="請選擇主題樣式" />
        </NFormItem>
        <NFormItem v-show="!cookiesEnable" path="customChatNum" label="聊天次數">
          <NInputNumber size="large" v-model:value="customChatNumSetting" min="0" style="width: 100%;"/>
        </NFormItem>
      </NForm>
      <template #action>
        <NButton size="large" @click="isShowAdvancedSettingModal = false">Cancel</NButton>
        <NButton ghost size="large" type="info" @click="saveAdvancedSetting">Save</NButton>
      </template>
    </NModal>
    <NModal v-model:show="isShowClearCacheModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-xl py-2">將要刪除包括 Cookie 在内的所有緩存？</div>
      </template>
      <template #action>
        <NButton size="large" @click="isShowClearCacheModal = false">Cancel</NButton>
        <NButton ghost size="large" type="error" @click="resetCache">Sure</NButton>
      </template>
    </NModal>
    <NModal v-model:show="isShowSetAboutModal" preset="dialog" :show-icon="false">
      <template #header>
        <div class="text-3xl py-2">About</div>
      </template>
      <NForm ref="formRef" label-placement="left" label-width="auto" size="small" style="margin-top: 16px;">
        <NFormItem path="" label="當前版本">
          <NTag type="info" size="small" round>{{ 'v' + localVersion }}</NTag>
        </NFormItem>
        <NFormItem path="" label="最新版本">
          <NTag type="info" size="small" round>{{ lastVersion }}</NTag>
        </NFormItem>
        <NFormItem path="token" label="開放源碼">
          <NButton text tag="a" :href="'https://github.com/'+author" target="_blank" type="success">{{ author }}</NButton>
        </NFormItem>
        <NFormItem path="token" label="原作者">
          <NButton text tag="a" href="https://github.com/adams549659584" target="_blank" type="success">adams549659584</NButton>
        </NFormItem>
        <NFormItem path="token" label="原始開源位址">
          <NButton text tag="a" href="https://github.com/adams549659584/go-proxy-bingai" target="_blank" type="success">adams549659584/go-proxy-bingai</NButton>
        </NFormItem>
      </NForm>
      <template #action>
        <NButton ghost size="large" @click="isShowSetAboutModal = false" type="info">確認</NButton>
      </template>
    </NModal>
  <CreateImage v-model:show="isShowCreateImageModal" />
</NConfigProvider></template>
