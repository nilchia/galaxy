<script setup lang="ts">
import { faUndo } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { BButton, BModal } from "bootstrap-vue";
import { ref } from "vue";

import { useActivityStore } from "@/stores/activityStore";

import ActivitySettings from "@/components/ActivityBar/ActivitySettings.vue";
import DelayedInput from "@/components/Common/DelayedInput.vue";
import ActivityPanel from "@/components/Panels/ActivityPanel.vue";

const props = defineProps<{
    activityBarId: string;
    heading: string;
    searchPlaceholder: string;
}>();

const emit = defineEmits<{
    (e: "activityClicked", activityId: string): void;
}>();

const activityStore = useActivityStore(props.activityBarId);

const confirmRestore = ref(false);
const query = ref("");

function onQuery(newQuery: string) {
    query.value = newQuery;
}
</script>

<template>
    <ActivityPanel :title="props.heading">
        <template v-slot:header>
            <DelayedInput :delay="100" :placeholder="props.searchPlaceholder" @change="onQuery" />
        </template>
        <template v-slot:header-buttons>
            <BButton
                v-b-tooltip.bottom.hover
                data-description="restore factory settings"
                size="sm"
                variant="link"
                title="Restore default"
                @click="confirmRestore = true">
                <span v-localize>Reset</span>
                <FontAwesomeIcon :icon="faUndo" fixed-width />
            </BButton>
        </template>
        <ActivitySettings
            :query="query"
            :activity-bar-id="props.activityBarId"
            @activityClicked="(...args) => emit('activityClicked', ...args)" />
        <BModal
            v-model="confirmRestore"
            title="Restore Activity Bar Defaults"
            title-tag="h2"
            @ok="activityStore.restore()">
            <p v-localize>Are you sure you want to reset the activity bar to its default settings?</p>
        </BModal>
    </ActivityPanel>
</template>
