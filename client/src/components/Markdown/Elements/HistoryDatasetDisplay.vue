<template>
    <b-card body-class="p-0">
        <b-card-header v-if="!embedded">
            <span class="float-right">
                <b-button
                    v-b-tooltip.hover
                    :href="downloadUrl"
                    variant="link"
                    size="sm"
                    role="button"
                    title="Download Dataset"
                    type="button"
                    class="py-0 px-1">
                    <span class="fa fa-download" />
                </b-button>
                <b-button
                    v-b-tooltip.hover
                    :href="importUrl"
                    role="button"
                    variant="link"
                    title="Import Dataset"
                    type="button"
                    class="py-0 px-1">
                    <span class="fa fa-file-import" />
                </b-button>
                <b-button
                    v-if="expanded"
                    v-b-tooltip.hover
                    href="#"
                    role="button"
                    variant="link"
                    title="Collapse"
                    type="button"
                    class="py-0 px-1"
                    @click="onExpand">
                    <span class="fa fa-angle-double-up" />
                </b-button>
                <b-button
                    v-else
                    v-b-tooltip.hover
                    href="#"
                    role="button"
                    variant="link"
                    title="Expand"
                    type="button"
                    class="py-0 px-1"
                    @click="onExpand">
                    <span class="fa fa-angle-double-down" />
                </b-button>
            </span>
            <span>
                <span>Dataset:</span>
                <span class="font-weight-light">{{ datasetName }}</span>
            </span>
        </b-card-header>
        <b-card-body>
            <UrlDataProvider v-slot="{ result: itemContent, loading: datasetLoading, error }" :url="itemUrl">
                <LoadingSpan v-if="datasetLoading" message="Loading Dataset" />
                <LoadingSpan v-else-if="loading" message="Loading Datatypes" />
                <div v-else-if="error">{{ error }}</div>
                <div v-else-if="!datatypesMapper">Datatypes not loaded.</div>
                <div v-else :class="contentClass">
                    <b-embed
                        v-if="datatypesMapper.isSubTypeOfAny(datasetType, ['pdf', 'html'])"
                        type="iframe"
                        aspect="16by9"
                        :src="displayUrl" />
                    <HistoryDatasetAsImage
                        v-else-if="datatypesMapper.isSubTypeOfAny(datasetType, ['galaxy.datatypes.images.Image'])"
                        :args="args" />
                    <div v-else-if="itemContent.item_data">
                        <div v-if="datatypesMapper.isSubTypeOfAny(datasetType, ['tabular'])">
                            <UrlDataProvider
                                v-slot="{ result: metaData, loading: metaLoading, error: metaError }"
                                :url="metaUrl">
                                <LoadingSpan v-if="metaLoading" message="Loading Metadata" />
                                <div v-else-if="metaError">{{ metaError }}</div>
                                <div v-else>
                                    <b-table
                                        id="tabular-dataset-table"
                                        sticky-header
                                        thead-tr-class="sticky-top"
                                        striped
                                        hover
                                        :per-page="perPage"
                                        :current-page="currentPage"
                                        :fields="getFields(metaData)"
                                        :items="getItems(itemContent.item_data, metaData)" />
                                    <b-pagination
                                        v-model="currentPage"
                                        align="center"
                                        :total-rows="getItems(itemContent.item_data, metaData).length"
                                        :per-page="perPage"
                                        aria-controls="tabular-dataset-table" />
                                </div>
                            </UrlDataProvider>
                        </div>
                        <pre v-else>
                            <code class="word-wrap-normal">{{ itemContent.item_data }}</code>
                        </pre>
                    </div>
                    <div v-else>No content found.</div>
                    <b-link v-if="itemContent.truncated" :href="itemContent.item_url"> Show More... </b-link>
                </div>
            </UrlDataProvider>
        </b-card-body>
    </b-card>
</template>

<script>
import LoadingSpan from "components/LoadingSpan";
import { UrlDataProvider } from "components/providers/UrlDataProvider";
import { getAppRoot } from "onload/loadConfig";
import { mapState } from "pinia";

import { useDatatypesMapperStore } from "@/stores/datatypesMapperStore";

import HistoryDatasetAsImage from "./HistoryDatasetAsImage.vue";

export default {
    components: {
        LoadingSpan,
        UrlDataProvider,
        HistoryDatasetAsImage,
    },
    props: {
        args: {
            type: Object,
            required: true,
        },
        datasets: {
            type: Object,
            required: true,
        },
        embedded: {
            type: Boolean,
            default: false,
        },
    },
    data() {
        return {
            currentPage: 1,
            expanded: false,
            perPage: 100,
        };
    },
    computed: {
        ...mapState(useDatatypesMapperStore, ["createMapper", "datatypesMapper", "loading"]),
        contentClass() {
            if (this.datatypesMapper?.isSubTypeOfAny(this.datasetType, ["tabular"])) {
                return "";
            }
            if (this.expanded) {
                return "embedded-dataset-expanded";
            } else {
                return "embedded-dataset";
            }
        },
        datasetType() {
            const dataset = this.datasets[this.args.history_dataset_id];
            return dataset.ext;
        },
        datasetName() {
            const dataset = this.datasets[this.args.history_dataset_id];
            return dataset && dataset.name;
        },
        downloadUrl() {
            return `${getAppRoot()}dataset/display?dataset_id=${this.args.history_dataset_id}`;
        },
        displayUrl() {
            return `${getAppRoot()}datasets/${this.args.history_dataset_id}/display/?preview=True`;
        },
        importUrl() {
            return `${getAppRoot()}dataset/imp?dataset_id=${this.args.history_dataset_id}`;
        },
        itemUrl() {
            return `/api/datasets/${this.args.history_dataset_id}/get_content_as_text`;
        },
        metaUrl() {
            return `/api/datasets/${this.args.history_dataset_id}`;
        },
    },
    created() {
        this.createMapper();
    },
    methods: {
        getFields(metaData) {
            const fields = [];
            const columnNames = metaData.metadata_column_names || [];
            const columnCount = metaData.metadata_columns;
            for (let i = 0; i < columnCount; i++) {
                fields.push({
                    key: `${i}`,
                    label: columnNames[i] || i,
                    sortable: true,
                });
            }
            return fields;
        },
        getItems(textData, metaData) {
            const tableData = [];
            const delimiter = metaData.metadata_delimiter || "\t";
            const comments = metaData.metadata_comment_lines || 0;
            const lines = textData.split("\n");
            lines.forEach((line, i) => {
                if (i >= comments) {
                    const tabs = line.split(delimiter);
                    const rowData = {};
                    let hasData = false;
                    tabs.forEach((cellData, j) => {
                        const cellDataTrimmed = cellData.trim();
                        if (cellDataTrimmed) {
                            hasData = true;
                        }
                        rowData[j] = cellDataTrimmed;
                    });
                    if (hasData) {
                        tableData.push(rowData);
                    }
                }
            });
            return tableData;
        },
        onExpand() {
            this.expanded = !this.expanded;
        },
    },
};
</script>
<style scoped>
.embedded-dataset {
    max-height: 20rem;
    overflow-y: auto;
}
.embedded-dataset-expanded {
    max-height: 40rem;
    overflow-y: auto;
}
</style>
