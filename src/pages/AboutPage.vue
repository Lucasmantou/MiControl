<script setup lang="ts">
import { onMounted, ref } from "vue";
import type { RuntimeSnapshot, ThemePreference } from "../lib/bridge";
import { getLaunchAtLogin, setLaunchAtLogin } from "../lib/bridge";
import { useTheme } from "../lib/theme";

defineProps<{ runtime: RuntimeSnapshot | null }>();

const {
  preference: themePreference,
  busy: themeBusy,
  errorMessage: themeError,
  setThemePreference,
} = useTheme();

const themeOptions: Array<{ value: ThemePreference; label: string }> = [
  { value: "system", label: "系统" },
  { value: "light", label: "浅色" },
  { value: "dark", label: "深色" },
];

const launchAtLogin = ref(false);
/**
 * 登录自启动的初始值来自异步 IPC。就绪前用同尺寸占位符顶位，就绪后才创建开关
 * 本体——元素"创建即带正确 checked"，从不存在属性变更，因此不会有关→开的滑动
 * 过渡。
 */
const launchAtLoginReady = ref(false);
const launchAtLoginBusy = ref(false);
const launchAtLoginError = ref("");

async function onThemeChange(event: Event): Promise<void> {
  await setThemePreference((event.target as HTMLInputElement).value as ThemePreference);
}

async function onLaunchAtLoginChange(event: Event): Promise<void> {
  const enabled = (event.target as HTMLInputElement).checked;
  launchAtLoginBusy.value = true;
  launchAtLoginError.value = "";
  try {
    launchAtLogin.value = await setLaunchAtLogin(enabled);
  } catch (error) {
    launchAtLoginError.value = error instanceof Error ? error.message : String(error);
  } finally {
    launchAtLoginBusy.value = false;
  }
}

onMounted(() => {
  void getLaunchAtLogin()
    .then((enabled) => { launchAtLogin.value = enabled; })
    .catch((error) => { launchAtLoginError.value = error instanceof Error ? error.message : String(error); })
    // 就绪标志与初值在同一渲染批次生效：开关此时才被创建，创建即带正确
    // checked，不产生属性变更，故无过渡可触发（无需禁用过渡或等待绘制）。
    .finally(() => { launchAtLoginReady.value = true; });
});
</script>

<template>
  <section>
    <header class="page-header">
      <div>
        <h1>关于</h1>
      </div>
    </header>

    <article class="card about-card">
      <img class="app-logo" src="/app-logo.png" alt="MiControl 应用图标" />
      <div>
        <h2>MiControl</h2>
        <p>版本 {{ runtime?.appVersion ?? "0.1.0" }}</p>
      </div>
    </article>

    <article class="card appearance-card">
      <h2>外观</h2>
      <p class="muted">选择应用的显示模式。</p>
      <div class="theme-selector" role="radiogroup" aria-label="显示模式">
        <label
          v-for="option in themeOptions"
          :key="option.value"
          class="theme-option"
          :class="{ selected: themePreference === option.value }"
        >
          <input
            type="radio"
            name="theme-preference"
            :value="option.value"
            :checked="themePreference === option.value"
            :disabled="themeBusy"
            @change="onThemeChange"
          />
          <span>{{ option.label }}</span>
        </label>
      </div>
      <p class="muted appearance-note">
        {{ themePreference === "system" ? "跟随 Windows 的应用颜色模式。" : "该选择会在重启后保持。" }}
      </p>
      <p v-if="themeError" class="error-text" role="alert">{{ themeError }}</p>
    </article>

    <article class="card">
      <h2>启动行为</h2>
      <p class="muted">登录 Windows 后自动启动 MiControl。</p>
      <label class="toggle-row" title="使用当前用户的 Windows 登录启动项，不需要管理员权限。">
        <input
          v-if="launchAtLoginReady"
          type="checkbox"
          class="toggle-input"
          name="launch-at-login"
          :checked="launchAtLogin"
          :disabled="launchAtLoginBusy"
          @change="onLaunchAtLoginChange"
        />
        <span v-else class="toggle-placeholder" aria-hidden="true"></span>
        登录时自动启动
      </label>
      <p v-if="launchAtLoginError" class="error-text" role="alert">{{ launchAtLoginError }}</p>
    </article>
  </section>
</template>

<style scoped>
/* 初值就绪前用同尺寸占位符顶位，避免开关出现时布局跳动；开关本体仅在
   终值就绪后创建，创建即带正确 checked，不产生关→开滑动过渡。 */
.toggle-placeholder { width: 34px; height: 20px; flex: none; }
</style>
