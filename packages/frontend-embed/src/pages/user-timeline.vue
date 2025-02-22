<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div>
	<EmLoading v-if="loading"/>
	<EmTimelineContainer v-else-if="user" :showHeader="embedParams.header">
		<template #header>
			<div :class="$style.userHeader">
				<a :href="`/@${user.username}`" target="_blank" rel="noopener noreferrer" :class="$style.avatarLink">
					<EmAvatar :class="$style.avatar" :user="user"/>
				</a>
				<div :class="$style.headerTitle">
					<I18n :src="i18n.ts.noteOf" tag="div" class="_nowrap">
						<template #user>
							<a v-if="user != null" :href="`/@${user.username}`" target="_blank" rel="noopener noreferrer">
								<EmUserName :user="user"/>
							</a>
							<span v-else>{{ i18n.ts.user }}</span>
						</template>
					</I18n>
					<div :class="$style.sub">{{ i18n.tsx.fromX({ x: instanceName }) }}</div>
				</div>
				<a :href="url" :class="$style.instanceIconLink" target="_blank" rel="noopener noreferrer">
					<img
						:class="$style.instanceIcon"
						:src="serverMetadata.iconUrl || '/favicon.ico'"
					/>
				</a>
			</div>
		</template>
		<template #body>
			<EmNotes
				ref="notesEl"
				:pagination="pagination"
				:disableAutoLoad="!embedParams.autoload"
				:noGap="true"
				:ad="false"
			/>
		</template>
	</EmTimelineContainer>
	<XNotFound v-else/>
</div>
</template>

<script setup lang="ts">
import { ref, computed, shallowRef, inject } from 'vue';
import * as Misskey from 'misskey-js';
import type { Paging } from '@/components/EmPagination.vue';
import EmNotes from '@/components/EmNotes.vue';
import EmAvatar from '@/components/EmAvatar.vue';
import EmLoading from '@/components/EmLoading.vue';
import EmUserName from '@/components/EmUserName.vue';
import I18n from '@/components/I18n.vue';
import XNotFound from '@/pages/not-found.vue';
import EmTimelineContainer from '@/components/EmTimelineContainer.vue';
import { misskeyApi } from '@/misskey-api.js';
import { i18n } from '@/i18n.js';
import { serverMetadata } from '@/server-metadata.js';
import { url, instanceName } from '@/config.js';
import { defaultEmbedParams } from '@@/js/embed-page.js';
import { DI } from '@/di.js';

const props = defineProps<{
	userId: string;
}>();

const embedParams = inject(DI.embedParams, defaultEmbedParams);

const user = ref<Misskey.entities.UserLite | null>(null);
const pagination = computed(() => ({
	endpoint: 'users/notes',
	params: {
		userId: user.value?.id,
	},
} as Paging));
const loading = ref(true);

const notesEl = shallowRef<InstanceType<typeof EmNotes> | null>(null);

misskeyApi('users/show', {
	userId: props.userId,
}).then(res => {
	user.value = res;
	loading.value = false;
}).catch(err => {
	console.error(err);
	loading.value = false;
});
</script>

<style lang="scss" module>
.userHeader {
	padding: 8px 16px;
	display: flex;
	min-width: 0;
	align-items: center;
	gap: var(--margin);
	overflow: hidden;

	.avatarLink {
		display: block;
	}

	.avatar {
		display: inline-block;
		width: 32px;
		height: 32px;
	}

	.headerTitle {
		flex-grow: 1;
		font-weight: 700;
		line-height: 1.1;
		min-width: 0;

		.sub {
			font-size: 0.8em;
			font-weight: 400;
			opacity: 0.7;
		}
	}

	.instanceIconLink {
		flex-shrink: 0;
		display: block;
		margin-left: auto;
		height: 24px;
	}

	.instanceIcon {
		height: 24px;
		border-radius: 4px;
	}
}
</style>
