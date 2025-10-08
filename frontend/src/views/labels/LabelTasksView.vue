<script setup lang="ts">
import {computed, shallowReactive, watch} from 'vue'
import {useRoute} from 'vue-router'

import {useAuthStore} from '@/stores/auth'
import {useLabelStore} from '@/stores/labels'

import LabelService from '@/services/label'
import LabelWrapper from '@/components/labels/LabelWrapper.vue'

import ProjectList from '@/components/project/views/ProjectList.vue'
import ProjectGantt from '@/components/project/views/ProjectGantt.vue'
import ProjectTable from '@/components/project/views/ProjectTable.vue'
import ProjectKanban from '@/components/project/views/ProjectKanban.vue'

const props = defineProps<{
	labelId: number,
}>()

const route = useRoute()
const authStore = useAuthStore()
const labelStore = useLabelStore()

const labelService = shallowReactive(new LabelService())
const isLoadingLabel = computed(() => labelService.loading)

const currentLabel = computed(() => labelStore.labels[props.labelId])

// Load label data
watch(
	() => props.labelId,
	async (labelIdToLoad) => {
		try {
			const loadedLabel = await labelService.get({id: labelIdToLoad})
			labelStore.setLabel(loadedLabel)
		} catch (error) {
			console.error('Error loading label:', error)
		}
	},
	{immediate: true},
)

// Determine which view to show based on user's default view setting
const currentViewKind = computed(() => {
	const defaultView = authStore.settings.frontendSettings.defaultView
	// 'first' means use the first available view, but for labels we default to list
	// Default to list view if no preference is set or if it's set to 'first'
	return (defaultView && defaultView !== 'first') ? defaultView : 'list'
})
</script>

<template>
	<LabelWrapper
		:is-loading-label="isLoadingLabel"
		:label-id="labelId"
	>
		<ProjectList
			v-if="currentViewKind === 'list'"
			:label-id="labelId"
			:project-id="0"
			:view-id="0"
			:is-loading-project="isLoadingLabel"
		/>
		<ProjectGantt
			v-if="currentViewKind === 'gantt'"
			:route
			:label-id="labelId"
			:is-loading-project="isLoadingLabel"
		/>
		<ProjectTable
			v-if="currentViewKind === 'table'"
			:label-id="labelId"
			:project-id="0"
			:view-id="0"
			:is-loading-project="isLoadingLabel"
		/>
		<ProjectKanban
			v-if="currentViewKind === 'kanban'"
			:label-id="labelId"
			:project-id="0"
			:view-id="0"
			:is-loading-project="isLoadingLabel"
		/>
	</LabelWrapper>
</template>
