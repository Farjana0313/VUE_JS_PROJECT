<template>
    <div class="h-full flex flex-col">
        <ListFilter
            ref="filter"
            :class="{ 'mb-8': hasFilters || search }"
            :filter="filter"
            :has-filters="hasFilters"
            :has-quick-filters="hasQuickFilters"
            :query="query"
            :search="search"
            :search-placeholder="searchPlaceholder"
            :filter-classes="filterClasses"
            :initial-values="initialFilter"
            :apply-filter="values => applyFilter(filterMapper(values))"
            :apply-search="applySearch"
            :clear-search="clearSearch"
        >
            <template #filters="props">
                <slot
                    name="filters"
                    :reset="props.reset"
                    :filter="filter"
                />
            </template>
            <template #quickfilter>
                <slot
                    name="quickfilter"
                    :apply-filter="applyFilter"
                />
            </template>
            <template #loader>
                <loader
                    :loading="loading"
                    :backdrop="true"
                />
            </template>
        </ListFilter>
        <!-- <div>
        </div> -->
        <vuetable
            ref="vuetable"
            :row-class="rowClass"
            :api-mode="false"
            :fields="columns"
            :data-manager="dataManager"
            :css="css.table"
            :track-by="trackBy"
            :sort-order="innerSortOrder"
            :detail-row-component="detailRow"
            :detail-row-options="detailOptions"
            :no-data-template="noDataTemplate"
            pagination-path="pagination"
            @vuetable:row-clicked="handleRowClicked"
            @vuetable:cell-clicked="props => $emit('cell-clicked', props)"
            @vuetable:pagination-data="onPaginationData"
        >
            <template #empty-result>
                <slot name="empty-result" />
            </template>
            <template #actions="props">
                <div v-if="hasActions">
                    <ListActions :record="props.rowData">
                        <slot
                            name="actions"
                            :record="props.rowData"
                        />
                    </ListActions>
                </div>
            </template>
            <template #inline-actions="props">
                <div v-if="hasInlineActions">
                    <InlineActions :record="props.rowData">
                        <slot
                            name="inline-actions"
                            :record="props.rowData"
                        />
                    </InlineActions>
                </div>
            </template>
            <template #detail-toggler="props">
                <div v-if="hasDetailRow">
                    <DetailToggler
                        :row-data="props.rowData"
                        :row-index="props.rowIndex"
                    >
                        <template #default="detailProps">
                            <slot
                                name="detail-toggler"
                                :record="props.rowData"
                                :is-open="detailProps.isOpen"
                                :toggle-row="detailProps.toggleRow"
                            />
                        </template>
                    </DetailToggler>
                </div>
            </template>
        </vuetable>
        <div
            v-if="!infinityScroll"
            class="pt-2"
        >
            <vuetable-pagination
                ref="pagination"
                :css="css.pagination"
                @vuetable-pagination:change-page="onChangePage"
            />
        </div>
    </div>
</template>
<script>
import { Vuetable, VuetablePagination } from 'vue3-vuetable';
import AuthMixin from '@/components/auth/AuthMixin';
import ModalNavigation from '@/mixins/ModalNavigation';
// import Loader from '@/components/ui/Loader';
import { throttle } from 'lodash-es';
import ListFilter from '@/components/auth/list/ListFilter';
import NotifyMixin from '@/mixins/NotifyMixin';
const css = {
    table: {
        tableClass: 'table-auto w-full table',
        tableBodyClass: '',
        tableHeaderClass: 'px-4 py-2',
        tableWrapper: 'overflow-x-auto flex-1',
        loadingClass: 'loading',
        ascendingIcon: 'blue chevron up icon',
        descendingIcon: 'blue chevron down icon',
        ascendingClass: 'sorted-asc',
        descendingClass: 'sorted-desc',
        sortableIcon: 'grey sort icon',
        handleIcon: 'grey sidebar icon',
        detailRowClass: 'bg-blue-100',
    },
    pagination: {
        wrapperClass: 'flex justify-center py-4',
        activeClass: 'active',
        disabledClass: '',
        pageClass: 'btn-paging',
        linkClass: 'btn-paging',
        paginationClass: '',
        paginationInfoClass: 'pagination-info',
        dropdownClass: '',
        icons: {
            first: '',
            prev: '',
            next: '',
            last: '',
        },
    },
};
export default {
    components: {
        // Loader,
        Vuetable,
        VuetablePagination,
        ListFilter,
    },
    mixins: [NotifyMixin, AuthMixin, ModalNavigation],
    props: {
        css: {
            type: Object,
            default() {
                return css;
            },
        },
    },
    data() {
        return {
            // loading: false,
            query: '',
            // filter: this.filterMapper(this.initialFilter),
            //sort: undefined,
            showFilters: false,
            loading: false,
            scrollContainer: null,
            innerSortOrder: this.sortOrder || [],
            data: [],
            sortOrder: [
                {
                    field: "first_name",
                    direction: "asc"
                }
            ],
            columns: [
                {
                    name: 'first_name',
                    title: 'First Name',
                    class: 'w-1/4',
                    sortField: 'first_name'
                
                },
                {
                    name: 'last_name',
                    title: 'Last Name',
                    class: 'w-1/4',
                    sortField: 'last_name'
                },
                {
                    name: 'email',
                    title: 'Email',
                    class: 'w-1/4',
                },
                {
                    name: 'phone_number',
                    title: 'Phone Number',
                    class: 'w-1/4',
                },
                {
                    name: 'communities',
                    title: 'Communities',
                    class: 'w-1/4',
                },
                {
                    label: 'communities',
                    field: 'Communities',
                    class: 'w-1/4',
                    formatter(value) {
                        // console.log('ROW', value);
                        let communities = [];
                        value.community_has_leasing_agents.forEach(function (row) {
                            communities.push('<span class="tag">' + row.community_id + '</span>');
                        });
                        // console.log('communities=>', communities);
                        return communities.join('');
                    },
                },
            ],
        };
    },
    computed: {
        search() {
            return true;
        },
        searchPlaceholder() {
            return 'Search by name, community, email';
        },
        hasFilters() {
            // return !!this.$slots.filters;
            return true;
        },
        hasQuickFilters() {
            // return !!this.$slots['quickfilter'];
            return true;
        },
        hasActions() {
            // return !!this.$slots.actions;
            return true;
        },
        hasInlineActions() {
            // return !!this.$slots['inline-actions'];
            return true;
        },
        hasDetailRow() {
            // return this.detailRow !== "";
            return true;
        },
        hasClickRowListener() {
            // return this.$attrs['row-clicked'];
            return true;
        },
        nonClickableRow() {
            return (
                this.onRowClick === 'none' &&
                !this.hasClickRowListener &&
                !(this.$route.params?.passFlowTo && this.$route.params?.passFlowTo !== this.$route.name)
            );
        },
        detailOptions() {
            return {
                ...this.detailRowOptions,
                vuetable: this.$refs.vuetable,
            };
        },
    },
    async mounted() {
        console.log('== Leasing Agents Index ==');
        console.log(this.profile);
        await this.getLeasingAgentsFromAPI();
    },
    created() {
        if (!this.community) {
            this.notifyError('please select a community to continue, then refresh the browser');
        }
    },
    methods: {
        async getLeasingAgentsFromAPI() {
            console.log('LEASING::getLeasingAgentsFromAPI()');
            this.loading = true;
            return await this.$calendarDataProvider
                .get('calendarGetLeasingAgents', {
                    customer_id: this.profile.customerId,
                })
                .then(res => {
                    console.log('LEASING::getLeasingAgentsFromAPI() result:');
                    console.log(res);
                    this.data = res;
                    return res;
                    // return this.getEvents(res);
                })
                .catch(err => console.log(err.message))
                .finally(() => {
                    this.loading = false;
                });
        },
        rowClass(dataItem) {
            const classes = ['table-row', 'row'];
            if (this.nonClickableRow) {
                classes.push('table-row-nonclickable');
            }
            if (this.hasDetailRow && this.$refs.vuetable?.isVisibleDetailRow(dataItem[this.trackBy])) {
                classes.push('table-row-detail-open');
            }
            return classes.join(' ');
        },
        toggleFilters() {
            this.showFilters = !this.showFilters;
        },
        applySearch(value) {
            this.query = value;
        },
        clearSearch() {
            this.query = '';
        },
        applyFilter(values) {
            this.filter = values;
            this.$refs.vuetable?.changePage(1);
            this.$refs.vuetable?.resetData();
        },
        doSearch: throttle(
            function () {
                if (this.query.length > 2 || this.query.length === 0) {
                    this.reload();
                }
            },
            500,
            { trailing: true }
        ),
        onPaginationData(paginationData) {
            if (!this.infinityScroll) {
                this.$refs.pagination.setPaginationData(paginationData);
            }
        },
        onChangePage(page) {
            this.$refs.vuetable.changePage(page);
        },
        reload() {
            this.$refs.vuetable.resetData();
            this.$refs.vuetable.refresh();
        },
        getSort({ sortField, direction }) {
            // sortField: 'type&bundle.name', - sort by two fields
            const fields = sortField.split('&');
            if (fields.length > 1) {
                return fields.map(item => {
                    const [fieldName, dir = direction] = item.split('|');
                    return `${fieldName},${dir}`;
                });
            }
            return `${sortField},${direction}`;
        },
        async dataManager(/*sortOrder = [], pagination = { current_page: 1 }*/) {
            // allow static data
            // if (this.data) {
            //     return {
            //         data: this.data,
            //     };
            // }
            const data = await this.getLeasingAgentsFromAPI();
            return {
                data: data,
            };
            // const prevData = this.$refs.vuetable?.tableData ?? [];
            // const { pageSize: size, query, filter, sort: oldSort } = this;
            // const sort = sortOrder[0] ? this.getSort(sortOrder[0]) : undefined;
            // this.loading = true;
            // return this[this.dataProvider]
            //     .getList(this.resource, {
            //         page: pagination.current_page - 1,
            //         size,
            //         sort,
            //         query,
            //         ...this.requestParams,
            //         ...filter,
            //     })
            //     .then(this.responseMapper)
            //     .then(({ content: nextData, totalElements }) => {
            //         const newPagination = this.$refs.vuetable?.makePagination(totalElements, this.pageSize, pagination.current_page);
            //         this.sort = sort;
            //         this.innerSortOrder = sortOrder;
            //         return {
            //             pagination: newPagination,
            //             data: this.infinityScroll && oldSort === sort ? [...prevData, ...nextData] : nextData,
            //         };
            //     })
            //     .catch(error => {
            //         this.notifyError(error.message);
            //         return {
            //             pagination: this.$refs.vuetable?.tablePagination,
            //             data: this.$refs.vuetable?.tableData,
            //         };
            //     })
            //     .finally(() => (this.loading = false));
        },
        handleRowClicked({ data, ...rest }) {
            if (this.$route.params?.passFlowTo && this.$route.params?.passFlowTo !== this.$route.name) {
                this.$router.push({
                    name: this.$route.params?.passFlowTo,
                    params: {
                        ...this.$route.params,
                        [this.routeIdParam]: data[this.trackBy],
                    },
                });
            } else {
                switch (this.onRowClick) {
                case 'edit':
                    this.$router.replace({ path: `${this.basePath}/${data[this.trackBy]}` });
                    break;
                case 'details':
                    this.$router.replace({ path: `${this.basePath}/${data[this.trackBy]}/details` });
                    break;
                default:
                    this.$emit('row-clicked', { data, ...rest });
                }
            }
        },
        handleScrollContainer(e) {
            if (this.$refs.vuetable.tablePagination && e.target.scrollHeight - e.target.scrollTop - e.target.clientHeight <= 2) {
                this.onChangePage('next');
            }
        },
    },
};
</script>
<style scoped>
.tag {
    display: inline-flex;
    align-items: center;
    overflow: hidden;
    margin-right: 0.25rem;
    padding-left: 0.5rem;
    padding-right: 0.5rem;
    padding-top: 0.25rem;
    padding-bottom: 0.25rem;
    border-radius: 0.0625rem;
    --bg-opacity: 1;
    background-color: #eaf6ff;
    background-color: rgba(234, 246, 255, var(--bg-opacity));
    --text-opacity: 1;
    color: #105d91;
    color: rgba(16, 93, 145, var(--text-opacity));
    font-family: 'FrankNew', sans-serif;
    font-weight: 500;
}
</style>