<template>
    <FormCard :title="outputTitle" collapsible :expanded.sync="expanded">
        <template v-slot:body>
            <FormOutputLabel :name="outputName" :step="step" />
            <FormElement
                :id="actionNames.RenameDatasetAction__newname"
                :value="formData[actionNames.RenameDatasetAction__newname]"
                :help="renameHelp"
                title="Rename dataset"
                type="text"
                @input="onInput" />
            <FormDatatype
                :id="actionNames.ChangeDatatypeAction__newtype"
                :value="formData[actionNames.ChangeDatatypeAction__newtype]"
                :datatypes="datatypes"
                title="Change datatype"
                help="This action will change the datatype of the output to the indicated datatype."
                @onChange="onDatatype" />
            <FormElement
                :id="actionNames.TagDatasetAction__tags"
                :value="formData[actionNames.TagDatasetAction__tags]"
                :attributes="{ placeholder: 'Enter Tags' }"
                title="Add Tags"
                type="tags"
                help="This action will set tags for the dataset."
                @input="onInput" />
            <FormElement
                :id="actionNames.RemoveTagDatasetAction__tags"
                :value="formData[actionNames.RemoveTagDatasetAction__tags]"
                :attributes="{ placeholder: 'Enter Tags' }"
                title="Remove Tags"
                type="tags"
                help="This action will remove tags for the dataset."
                @input="onInput" />
            <FormCard title="Assign columns" collapsible :expanded.sync="expandedColumn">
                <template v-slot:body>
                    <FormElement
                        :id="actionNames.ColumnSetAction__chromCol"
                        :value="formData[actionNames.ColumnSetAction__chromCol]"
                        title="Chrom column"
                        type="integer"
                        help="This action will set the chromosome column."
                        @input="onInput" />
                    <FormElement
                        :id="actionNames.ColumnSetAction__startCol"
                        :value="formData[actionNames.ColumnSetAction__startCol]"
                        title="Start column"
                        type="integer"
                        help="This action will set the start column."
                        @input="onInput" />
                    <FormElement
                        :id="actionNames.ColumnSetAction__endCol"
                        :value="formData[actionNames.ColumnSetAction__endCol]"
                        title="End column"
                        type="integer"
                        help="This action will set the end column."
                        @input="onInput" />
                    <FormElement
                        :id="actionNames.ColumnSetAction__strandCol"
                        :value="formData[actionNames.ColumnSetAction__strandCol]"
                        title="Strand column"
                        type="integer"
                        help="This action will set the strand column."
                        @input="onInput" />
                    <FormElement
                        :id="actionNames.ColumnSetAction__nameCol"
                        :value="formData[actionNames.ColumnSetAction__nameCol]"
                        title="Name column"
                        type="integer"
                        help="This action will set the name column."
                        @input="onInput" />
                </template>
            </FormCard>
        </template>
    </FormCard>
</template>

<script>
import FormCard from "@/components/Form/FormCard";
import FormElement from "@/components/Form/FormElement";
import FormDatatype from "@/components/Workflow/Editor/Forms/FormDatatype";
import FormOutputLabel from "@/components/Workflow/Editor/Forms/FormOutputLabel";

const actions = [
    "RenameDatasetAction__newname",
    "ChangeDatatypeAction__newtype",
    "TagDatasetAction__tags",
    "RemoveTagDatasetAction__tags",
];

const actionsColumn = [
    "ColumnSetAction__chromCol",
    "ColumnSetAction__startCol",
    "ColumnSetAction__endCol",
    "ColumnSetAction__strandCol",
    "ColumnSetAction__nameCol",
];

export default {
    components: {
        FormCard,
        FormElement,
        FormOutputLabel,
        FormDatatype,
    },
    props: {
        outputName: {
            type: String,
            required: true,
        },
        outputLabel: {
            type: String,
            default: null,
        },
        inputs: {
            type: Array,
            required: true,
        },
        datatypes: {
            type: Array,
            required: true,
        },
        formData: {
            type: Object,
            required: true,
        },
        step: {
            // type Step from @/stores/workflowStepStore
            type: Object,
            required: true,
        },
    },
    data() {
        return {
            expanded: false,
            expandedColumn: false,
            renameHelpUrl: "https://galaxyproject.org/learn/advanced-workflow/variables/",
        };
    },
    computed: {
        outputTitle() {
            const title = this.outputLabel || this.outputName;
            return `Configure Output: '${title}'`;
        },
        actionNames() {
            const index = {};
            actions.concat(actionsColumn).forEach((action) => {
                index[action] = `pja__${this.outputName}__${action}`;
            });
            return index;
        },
        renameHelp() {
            /* TODO: FormElement should provide a slot for custom help templating instead. */
            const helpLink = `<a href="${this.renameHelpUrl}">here</a>`;
            const helpSection = `This action will rename the output dataset. Click ${helpLink} for more information. Valid input variables are:`;
            let helpLabels = "";
            for (const input of this.inputs) {
                const name = input.name.replace(/\|/g, ".");
                const label = input.label ? `(${input.label})` : "";
                helpLabels += `<li><strong>${name}</strong>${label}</li>`;
            }
            return `${helpSection}<ul>${helpLabels}</ul>`;
        },
    },
    created() {
        this.setExpanded();
    },
    methods: {
        setExpanded() {
            this.expandedColumn = this.hasActions(actionsColumn);
            if (this.expandedColumn) {
                this.expanded = true;
            } else {
                this.expanded = this.hasActions(actions);
            }
        },
        hasActions(actions) {
            for (const key of actions) {
                if (this.formData[this.actionNames[key]] != undefined) {
                    return true;
                }
            }
            return false;
        },
        onInput(value, pjaKey) {
            this.$emit("onInput", value, pjaKey);
        },
        onDatatype(newDatatype) {
            this.$emit("onDatatype", this.actionNames.ChangeDatatypeAction__newtype, this.outputName, newDatatype);
        },
    },
};
</script>
