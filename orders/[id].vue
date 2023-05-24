<script setup>
    import {ref} from 'vue';
    import { storeToRefs } from 'pinia';
    import { useTypeView } from "~/stores/view-type";
    import { usePersonalAccount } from "~/stores/personal-account";
    import api from '~/api';
    import { useI18n } from "vue-i18n";
    import {getOrderStatus} from '~/modules/OrderStatus';

    import OrderOption from '~/components/PersonalAccount/OrderOption';
    import OrderContent from '~/components/Account/OrderContent';

    const { isMobile } = useDevice();
    const viewTypeStore = useTypeView();
    const { viewType } = storeToRefs(viewTypeStore);
    const personalAccount = usePersonalAccount();
    const {id} = useRoute().params;
    const isShow = ref(false);
    const { t } = useI18n();

    const order = ref({});

    const options = [
        [
            {key: 'date', label: 'Дата заказа', viewType: 'row'},
            {key: 'state', label: 'Статус', viewType: 'row'}
        ],
        [
            {key: 'address', label: 'Адрес доставки'},
            {key: 'delivery_type', label: 'Способ доставки'},
            {key: 'paymentType', label: 'Форма оплаты'}
        ]
    ];

    const renderAddress = () => {
        try {
            if(order.value.delivery_type === 'courier' && order.value.address) {
                return `${order.value.address.street} ${order.value.address.building}, ${order.value.address.flat}`;
            }

            if(order.value.delivery_type === 'pick_up' && order.value.point) {
                return `${order.value.point.address.street} ${order.value.point.address.building}, ${t('кв')} ${order.value.point.address.flat}`;
            }

            return '-';

        } catch(err) {
            console.error(err);
            return '-';
        }
    }

    const run = async() => {
        try {
            const dataOrder = await api.account.getOrder(id);

            order.value = dataOrder.data;
            isShow.value = true;

        } catch(err) {
            console.error(err);
        }
    }

    run();

    definePageMeta({
        layout: 'account-layout',
    });
</script>

<template>
    <div class="order_history section-effect"
         :class="{'active': isShow}"
    >
        <div v-if="isShow">
            <div class="account_container"
                 v-if="!isMobile && viewType === 'desktop'"
            >
                <h2 class="order_history-title font-bold">{{ $t('Заказ') }}№ {{ order.reference }}</h2>
                <NuxtLink :to="localePath('/account/orders')"
                          class="order_history-link bee-link font-bold order_history-link--top"
                >
                    <div class="flex items-center">
                        <i class="order_history-link-icon icon icon-24" />
                        <span>{{ $t('Вернуться к списку заказов') }}</span>
                    </div>
                </NuxtLink>

                <div calss="order_history-info">
                    <div v-for="(row, index) in options" :key="index"
                         class="grid order_history-grid"
                         :class="'grid-cols-3'"
                    >
                        <OrderOption v-for="option in row" :key="option.key"
                                     :option="option"
                                     :order="order"
                        />
                    </div>
                </div>

                <OrderContent :order="order" />

                <NuxtLink :to="localePath('/account/orders')" class="order_history-link bee-link font-bold">
                    <div class="flex items-center">
                        <i class="order_history-link-icon icon icon-24" />
                        <span>{{ $t('Вернуться к списку заказов') }}</span>
                    </div>
                </NuxtLink>
            </div>

            <div v-if="isMobile || viewType === 'mobile'">
                <div class="order_history-props">
                    <div class="grid grid-cols-2 order_history-props-grid">
                        <div class="form-group">
                            <label class="w-full font-bold">{{$t('Дата заказа')}}</label>
                            <p>{{ order.date }}</p>
                        </div>
                        <div class="form-group">
                            <label class="w-full font-bold">{{$t('Статус заказа')}}</label>
                            <p>{{ $t(getOrderStatus(order.state)) }}</p>
                        </div>
                    </div>

                    <div class="grid grid-cols-1 order_history-props-grid">
                        <label class="w-full font-bold">{{$t('Адрес доставки')}}</label>
                        <p>{{ renderAddress() }}</p>
                    </div>

                    <div class="grid grid-cols-1 order_history-props-grid">
                        <label class="w-full font-bold">{{$t('Способ доставки')}}</label>
                        <p>
                            {{ order.delivery_type === 'pick_up' ? t('Курьер по вашему адресу') : t('Самовывоз из салона') }}
                        </p>
                    </div>

                    <div class="grid grid-cols-1">
                        <label class="w-full font-bold">{{$t('Форма оплаты')}}</label>
                        <p>{{$t('Картой или наличными при получении товара')}}</p>
                    </div>

                    <OrderContent :order="order" />
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="less">
    .order_history {
        &-title {
            margin-bottom: 34px;
        }

        & .order_history-grid {
            margin-bottom: 40px;
        }

        &-link {
            position: relative;
            left: -8px;
            display: inline-block;

            &--top {
                margin-bottom: 26px;
            }

            &-icon {
                transform: rotate(180deg);
                background-image: url('/images/pagination-arrow-icon-effect.svg');
            }

            &:hover, &:active {
                & .order_history-link-icon {
                    background-image: url('/images/pagination-arrow-icon.svg');
                }
            }
        }

        & .order_card {
            box-shadow: none;
        }
    }

    .order_history-props {
        &-grid {
            margin-bottom: 1rem;
        }

        & label {
            font-size: 16px;
        }

        & p {
            font-size: 14px;
        }
    }

    .app.app--mobile {
        & .order_history-props-grid {
            @media all and (max-width: 359px) {
                grid-template-columns: auto;


                & .form-group:first-child {
                    margin-bottom: 1rem;
                }
            }
        }
    }
</style>