<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<MkPagination :pagination="pagination">
	<template #empty><MkResult type="empty" :text="i18n.ts.noUsers"/></template>

	<template #default="{ items }">
		<div :class="$style.root">
			<MkUserInfo v-for="item in items" :key="item.id" class="user" :user="extractor(item)"/>
		</div>
	</template>
</MkPagination>
</template>

<script lang="ts" setup>
import type { PagingCtx } from '@/composables/use-pagination.js';
import MkUserInfo from '@/components/MkUserInfo.vue';
import MkPagination from '@/components/MkPagination.vue';
import { i18n } from '@/i18n.js';

const props = withDefaults(defineProps<{
	pagination: PagingCtx;
	noGap?: boolean;
	extractor?: (item: any) => any;
}>(), {
	extractor: (item) => item,
});
</script>

<style lang="scss" module>
.root {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
	grid-gap: var(--MI-margin);
}
</style>
