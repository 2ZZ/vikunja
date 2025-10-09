<script setup lang="ts">
import {computed, ref, shallowReactive, watch} from 'vue'
import {useRoute} from 'vue-router'

import {useAuthStore} from '@/stores/auth'
import {useLabelStore} from '@/stores/labels'

import LabelService from '@/services/label'
import LabelWrapper from '@/components/labels/LabelWrapper.vue'
import FilterPopup from '@/components/project/partials/FilterPopup.vue'

import ProjectList from '@/components/project/views/ProjectList.vue'
import ProjectTable from '@/components/project/views/ProjectTable.vue'

import {getDefaultTaskFilterParams, type TaskFilterParams} from '@/services/taskCollection'

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
// Note: Only List and Table views are currently supported for label filtering
// Gantt and Kanban require project context and are not yet supported
const currentViewKind = ref<'list' | 'table'>('list')

// Initialize view from user's default setting
watch(
	() => authStore.settings?.frontendSettings?.defaultView,
	(defaultView) => {
		// Only List and Table views work with label filtering
		if (defaultView === 'table') {
			currentViewKind.value = 'table'
		} else {
			currentViewKind.value = 'list'
		}
	},
	{immediate: true},
)

// Filter params for the FilterPopup
const filterParams = ref<TaskFilterParams>(getDefaultTaskFilterParams())
</script>

<template>
	<LabelWrapper
		:is-loading-label="isLoadingLabel"
		:label-id="labelId"
		:current-view-kind="currentViewKind"
		@update:view-kind="currentViewKind = $event"
	>
		<template #header>
			<div class="filter-container">
				<FilterPopup
					v-model="filterParams"
					:project-id="0"
					:view-id="0"
				/>
			</div>
		</template>

		<ProjectList
			v-if="currentViewKind === 'list'"
			:label-id="labelId"
			:project-id="0"
			:view-id="0"
			:is-loading-project="isLoadingLabel"
		/>
		<ProjectTable
			v-else-if="currentViewKind === 'table'"
			:label-id="labelId"
			:project-id="0"
			:view-id="0"
			:is-loading-project="isLoadingLabel"
		/>
	</LabelWrapper>
</template>
