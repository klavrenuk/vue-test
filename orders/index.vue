<script setup>
    import {getOrderStatus} from '~/modules/OrderStatus';
    import {ref} from 'vue';
    import { useDebounceFn } from "@vueuse/core";
    import api from '~/api';
    import { useTypeView } from "~~/stores/view-type";
    import { storeToRefs } from 'pinia'
    import dayjs from 'dayjs';

    import OrderHistoryTableHeader from '~/components/PersonalAccount/OrderHistoryTableHeader';
    import PaginationArrowNumbers from '~/components/Pagination/PaginationArrowNumbers';
    import AccountEmpty from '~/components/Account/AccountEmpty';
    import OrderCardMobile from '~/components/PersonalAccount/OrderCardMobile.vue';
    import ButtonComment from '~/components/Account/ButtonComment';

    const { isMobile } = useDevice();
    const viewTypeStore = useTypeView();
    const { viewType } = storeToRefs(viewTypeStore);
    const { t } = useI18n();
    const isShow = ref(false);

    const searchStr = ref('');
    const ordersAll = ref([]);
    const orders = ref([]);
    const pagination = ref({
        currentPage: 1,
        totalPages: 2
    });
    const currentPage = ref(1);
    const tableHeaders = [
        {key: 'reference', text: 'Заказ', isSort: false},
        {key: 'date', text: 'Дата', isSort: true},
        {key: 'total_amount', text: 'Сумма', isSort: true},
        {key: 'state', text: 'Статус', isSort: true},
        {key: 'comments', text: 'Отзыв', isSort: false},
    ];
    const viewTypePage = ref('table');

    const onSearch = useDebounceFn(() => {
        pagination.value.currentPage = 1;
        currentPage.value = 1;
        run(false);
    }, 450);

    const setSort = (type) => console.log('Sorted', key, type);

    const onChangePage = (activePage = 1) => {
        pagination.value.currentPage = activePage;
        run(false);
    }

    const onChangePageMobile = (activePage = 1) => {
        pagination.value.currentPage = activePage;
        run(false, true);
    }

    const toggleViewType = (type) => viewTypePage.value = type;

    const run = async(isInit = false, isMobileView = false) => {
        try {
            if(!isMobileView) isShow.value = false;

            const dataOrder = await api.account.getOrders(searchStr.value, pagination.value.currentPage);

            if(isInit) {
                if(!isMobile) ordersAll.value = dataOrder.data.slice();
                else ordersAll.value = ordersAll.value.concat(dataOrder.data);
            }

            pagination.value = {
                currentPage: dataOrder.pagination.current_page,
                totalPages: dataOrder.pagination.last_page
            };
            currentPage.value = dataOrder.pagination.current_page;

            if(!isMobileView) orders.value = dataOrder.data;
            else orders.value = orders.value.concat(dataOrder.data);

            isShow.value = true;

        } catch(err) {
            if(isInit) {
                ordersAll.value = [];
            }

            orders.value  = [];
            console.error(err);
        }
    }

    const renderValue = (key, order) => {
        switch (key) {
            case 'state':
                return t(getOrderStatus(order[key]));

            case 'date':
                const date = dayjs(order.date);
                return date.format('DD.MM.YYYY, hh:mm');

            case 'total_amount':
                if(order.total_amount) {
                    return order.total_amount.toLocaleString('ru') + ' ' + t('сум');
                } else {
                    return '0';
                }

            default:
                return order[key];
        }
    }

    run(true);

    definePageMeta({
        layout: 'account-layout',
    });
</script>

<template>
    <section class="history_orders section-effect"
             :class="{'active': isShow}"
    >
        <div v-if="isShow">
            <div class="account_container">
                <h2 class="history_orders-title font-bold">
                    {{ $t('Личные данные') }}
                </h2>

                <input class="history_orders-search bee-input"
                       v-model="searchStr"
                       @keypress.enter="onSearch"
                       @input="onSearch"
                       :placeholder="$t('Поиск по товару или номеру заказа') + '...'"
                       v-if="ordersAll[0]"
                />
            </div>

            <div class="history_orders-content w-full">
                <div v-if="ordersAll[0]">
                    <div v-if="!isMobile && viewType === 'desktop'">
                        <div class="history_orders-list">
                            <table class="table order_table w-full">
                                <OrderHistoryTableHeader
                                        :headers="tableHeaders"
                                        :setSort="setSort"
                                />

                                <tbody>
                                <tr v-for="order in orders" :key="order.id">
                                    <td v-for="header in tableHeaders"
                                        :key="`order-${order.id}-${header.key}`"
                                        class="table_cell"
                                        :class="'table_cell--' + header.key"
                                    >
                                        <NuxtLink
                                                :to="localePath('/account/orders/' + order.id)"
                                                class="bee-link"
                                                v-if="header.key === 'reference'"
                                        >
                                            {{  order[header.key]  }}
                                        </NuxtLink>

                                        <span v-if="header.key !== 'reference' && header.key !== 'comments'">
                                            {{ renderValue(header.key, order)}}
                                        </span>

                                        <span v-if="header.key === 'comments'">
                                            <ButtonComment />
                                        </span>
                                    </td>
                                </tr>
                                </tbody>
                            </table>
                        </div>

                        <PaginationArrowNumbers
                                :onChangePage="onChangePage"
                                :lastPage="pagination.totalPages"
                                :currentPage="currentPage"
                        />
                    </div>

                    <div v-if="isMobile || viewType === 'mobile'">
                        <ul class="list_orders_mobile list">
                            <li v-for="order in orders" :key="order.id"
                                class="list-item"
                            >
                                <OrderCardMobile :order="order" />
                            </li>
                        </ul>

                        <PaginationArrowNumbers
                                :onChangePage="onChangePageMobile"
                                :lastPage="pagination.totalPages"
                                :currentPage="currentPage"
                        />
                    </div>
                </div>

                <AccountEmpty v-if="!ordersAll[0]" type="orders" />
            </div>
        </div>
    </section>
</template>

<style lang="less">
    .history_orders {
        &-title {
            margin-bottom: 24px;
            font-family: "Druk-Text-Wide-Cyr",serif;
        }

        &-search {
            width: 100%;
            max-width: 456px;
            margin-bottom: 1rem;
            font-size: 16px;
        }

        & .order_table {
            & .table_cell {
                padding-left: 28px;
                padding-right: 28px;
                font-size: 16px;
                line-height: 18px;
            }
        }

        &-pagination {
            margin-top: 26px;
        }
    }

    .app.app--mobile {
        & .history_orders-title {
            font-size: 20px;
        }

        & .list_orders_mobile {
            & li {
                margin-bottom: .5rem;

                &:last-child {
                    margin-bottom: 0;
                }
            }
        }
    }
</style>