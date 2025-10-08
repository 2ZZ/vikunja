<template>
	<div
		class="loader-container"
		:class="{'is-loading': isLoadingLabel}"
	>
		<div class="switch-view-container">
			<h1 v-if="currentLabel" class="title">
				<span
					class="label-color-bubble"
					:style="{'background-color': currentLabel.hexColor}"
				/>
				{{ currentLabel.title }}
			</h1>
			<slot name="header" />
		</div>

		<slot v-if="!isLoadingLabel" />
	</div>
</template>

<script setup lang="ts">
import {computed} from 'vue'
import {useLabelStore} from '@/stores/labels'
import type {ILabel} from '@/modelTypes/ILabel'

const props = defineProps<{
	isLoadingLabel: boolean,
	labelId: ILabel['id'],
}>()

const labelStore = useLabelStore()

const currentLabel = computed(() => labelStore.labels[props.labelId])
</script>

<style scoped lang="scss">
.label-color-bubble {
	display: inline-block;
	width: 1em;
	height: 1em;
	border-radius: 50%;
	margin-right: 0.5em;
	vertical-align: middle;
}

.switch-view-container {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 1rem;
}
</style>
