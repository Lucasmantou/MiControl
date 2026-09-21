<script setup lang="ts">
import type { NavIcon, PageId } from "../navigation";
import { navigationItems } from "../navigation";

defineProps<{ activePage: PageId }>();
const emit = defineEmits<{ select: [page: PageId] }>();

/**
 * 底部导航图标：SVG path 组（24x24 视窗，描边风格）。
 * keyboard 对齐 macOS SF "keyboard"；settings 为齿轮（圆心 + 齿圈轮廓）。
 */
const ICON_PATHS: Record<NavIcon, { strokes: string[] }> = {
  keyboard: {
    // SF "keyboard"：圆角键盘轮廓 + 功能行点阵 + 底部长条
    strokes: [
      "M3.2 6.8h17.6a1.7 1.7 0 0 1 1.7 1.7v7a1.7 1.7 0 0 1-1.7 1.7H3.2a1.7 1.7 0 0 1-1.7-1.7v-7a1.7 1.7 0 0 1 1.7-1.7z",
      "M7 10.2h.01M10.4 10.2h.01M13.8 10.2h.01M17.2 10.2h.01",
      "M7.6 13.6h8.8",
    ],
  },
  settings: {
    // 齿轮：中心圆 + 外齿圈（单条连续轮廓）
    strokes: [
      "M12 15a3 3 0 1 0 0-6 3 3 0 0 0 0 6z",
      "M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z",
    ],
  },
};
</script>

<template>
  <nav class="bottom-nav" aria-label="设置页面">
    <button
      v-for="item in navigationItems"
      :key="item.id"
      class="nav-item"
      :class="{ active: activePage === item.id }"
      type="button"
      @click="emit('select', item.id)"
    >
      <svg class="nav-icon" viewBox="0 0 24 24" fill="none" aria-hidden="true">
        <path
          v-for="(path, index) in ICON_PATHS[item.icon].strokes"
          :key="index"
          :d="path"
          stroke="currentColor"
          stroke-width="1.9"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
      </svg>
      <span>{{ item.label }}</span>
    </button>
  </nav>
</template>
