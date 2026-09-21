<script setup lang="ts">
import type { RuntimeSnapshot } from "../lib/bridge";
import AboutPage from "./AboutPage.vue";
import ConnectionPage from "./ConnectionPage.vue";
import PermissionsPage from "./PermissionsPage.vue";

defineProps<{ runtime: RuntimeSnapshot | null }>();
</script>

<template>
  <div class="settings-page">
    <ConnectionPage :runtime="runtime" />
    <PermissionsPage :runtime="runtime" />
    <AboutPage :runtime="runtime" />
  </div>
</template>
