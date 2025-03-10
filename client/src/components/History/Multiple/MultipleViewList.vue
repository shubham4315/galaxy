<template>
    <div class="list-container d-flex h-100 w-auto overflow-auto">
        <virtual-list
            v-if="selectedHistories.length"
            :estimate-size="selectedHistories.length"
            :data-key="'id'"
            :data-component="MultipleViewItem"
            :data-sources="selectedHistories"
            :direction="'horizontal'"
            :extra-props="{ currentHistory, handlers, filter, removeHistoryFromList }"
            :item-style="{ width: '15rem' }"
            item-class="d-flex mx-1 mt-1"
            class="d-flex"
            wrap-class="row flex-nowrap m-0">
        </virtual-list>

        <div
            v-b-modal.select-histories-modal
            class="history-picker text-primary d-flex m-3 p-5 align-items-center text-nowrap">
            Select histories
        </div>

        <SelectorModal
            id="select-histories-modal"
            :multiple="true"
            :histories="histories"
            :current-history-id="currentHistory.id"
            title="Select histories"
            @selectHistories="addHistoriesToList" />
    </div>
</template>

<script>
import { mapActions, mapGetters } from "vuex";
import VirtualList from "vue-virtual-scroll-list";
import MultipleViewItem from "./MultipleViewItem";
import SelectorModal from "components/History/Modals/SelectorModal";

export default {
    components: {
        VirtualList,
        SelectorModal,
    },
    props: {
        histories: {
            type: Array,
            required: true,
        },
        currentHistory: {
            type: Object,
            required: true,
        },
        handlers: {
            type: Object,
            required: true,
        },
        filter: {
            type: String,
            default: "",
        },
    },
    data() {
        return {
            MultipleViewItem,
        };
    },
    computed: {
        ...mapGetters("history", ["getPinnedHistories"]),
        selectedHistories() {
            return this.getPinnedHistories();
        },
    },
    created() {
        if (!this.selectedHistories.length) {
            const firstHistory = this.histories[0];
            this.pinHistory(firstHistory.id);
        }
    },
    methods: {
        ...mapActions("history", ["pinHistory", "unpinHistory"]),
        addHistoriesToList(histories) {
            histories.forEach((history) => {
                const historyExists = this.selectedHistories.find((h) => h.id == history.id);
                if (!historyExists) {
                    this.pinHistory(history.id);
                }
            });
        },
        removeHistoryFromList(history) {
            this.unpinHistory(history.id);
        },
    },
};
</script>

<style lang="scss">
.list-container {
    width: fit-content;

    .history-picker {
        border: dotted lightgray;
    }

    background-image: linear-gradient(to right, white, white), linear-gradient(to right, white, white),
        linear-gradient(to right, rgba(0, 0, 0, 0.25), rgba(255, 255, 255, 0)),
        linear-gradient(to left, rgba(0, 0, 0, 0.25), rgba(255, 255, 255, 0));
    background-position: left center, right center, left center, right center;
    background-repeat: no-repeat;
    background-color: white;
    background-size: 20px 100%, 20px 100%, 10px 100%, 10px 100%;
    background-attachment: local, local, scroll, scroll;
}
</style>
