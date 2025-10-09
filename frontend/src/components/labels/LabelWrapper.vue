<template>
	<div
		class="loader-container"
		:class="{'is-loading': isLoadingLabel}"
	>
		<h1 v-if="currentLabel" class="label-title">
			{{ currentLabel.title }}
		</h1>

		<div
			class="switch-view-container d-print-none"
			:class="{'is-justify-content-flex-end': availableViews.length === 1}"
		>
			<div
				v-if="availableViews.length > 1"
				class="switch-view"
			>
				<BaseButton
					v-for="view in availableViews"
					:key="view.kind"
					class="switch-view-button"
					:class="{'is-active': view.kind === currentViewKind}"
					@click="$emit('update:viewKind', view.kind)"
				>
					{{ view.title }}
				</BaseButton>
			</div>
			<slot name="header" />
		</div>

		<slot v-if="!isLoadingLabel" />
	</div>
</template>

<script setup lang="ts">
import {computed} from 'vue'
import {useI18n} from 'vue-i18n'
import {useLabelStore} from '@/stores/labels'
import BaseButton from '@/components/base/BaseButton.vue'
import type {ILabel} from '@/modelTypes/ILabel'

const props = defineProps<{
	isLoadingLabel: boolean,
	labelId: ILabel['id'],
	currentViewKind: 'list' | 'table',
}>()

defineEmits<{
	'update:viewKind': [kind: 'list' | 'table']
}>()

const {t} = useI18n()
const labelStore = useLabelStore()

const currentLabel = computed(() => labelStore.labels[props.labelId])

const availableViews = [
	{kind: 'list' as const, title: t('project.list.title')},
	{kind: 'table' as const, title: t('project.table.title')},
]
</script>

<style scoped lang="scss">
.label-title {
	margin-bottom: 1rem;
	font-size: 2rem;
	font-weight: bold;
	text-align: center;
}

.switch-view-container {
	min-block-size: $switch-view-height;
	margin-block-end: 1rem;

	display: flex;
	justify-content: space-between;
	align-items: center;
	gap: 1rem;

	@media screen and (max-width: $tablet) {
		justify-content: center;
		flex-direction: column;
	}
}

.switch-view {
	background: var(--white);
	display: inline-flex;
	border-radius: $radius;
	font-size: .75rem;
	box-shadow: var(--shadow-sm);
	padding: .5rem;
}

.switch-view-button {
	padding: .25rem .5rem;
	display: block;
	border-radius: $radius;
	transition: all 100ms;

	&:not(:last-child) {
		margin-inline-end: .5rem;
	}

	&:hover {
		color: var(--switch-view-color);
		background: var(--primary);
	}

	&.is-active {
		color: var(--switch-view-color);
		background: var(--primary);
		font-weight: bold;
		box-shadow: var(--shadow-xs);
	}
}
</style>
