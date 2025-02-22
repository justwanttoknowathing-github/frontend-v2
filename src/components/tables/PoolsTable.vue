<template>
  <BalCard shadow="lg" class="mt-4" no-pad>
    <BalTable
      :columns="columns"
      :data="data"
      :is-loading="isLoading || isLoadingBalances"
      :is-loading-more="isLoadingMore"
      skeleton-class="h-64"
      sticky="both"
      :link="{
        to: 'pool',
        getParams: pool => ({ id: pool.id })
      }"
      :on-row-click="handleRowClick"
      :is-paginated="isPaginated"
      @load-more="$emit('loadMore')"
      :initial-state="{
        sortColumn: 'poolValue',
        sortDirection: 'desc'
      }"
    >
      <template v-slot:iconColumnHeader>
        <div class="flex items-center">
          <img :src="require('@/assets/images/icons/tokens.svg')" />
        </div>
      </template>
      <template v-slot:iconColumnCell="pool">
        <div v-if="!isLoading" class="px-6 py-4">
          <BalAssetSet
            :addresses="sortedTokenAddressesFor(pool)"
            :width="100"
          />
        </div>
      </template>
      <template v-slot:poolNameCell="pool">
        <div
          v-if="!isLoading && !isLoadingBalances"
          class="px-6 py-4 -mt-1 flex flex-wrap"
        >
          <div
            v-for="token in sortedTokensFor(pool)"
            :key="token"
            class="flex items-center px-2 mr-2 my-1 py-1 rounded-lg bg-gray-50 relative"
          >
            <div
              v-if="hasBalance(token.address)"
              class="w-3 h-3 rounded-full border-2 border-white hover:border-gray-50 bg-green-200 absolute top-0 right-0 -mt-1 -mr-1"
            />
            <span>
              {{ allTokens[getAddress(token.address)]?.symbol }}
            </span>
            <span class="font-medium text-gray-400 text-xs mt-px ml-1">
              {{ fNum(token.weight, 'percent_lg') }}
            </span>
          </div>
        </div>
      </template>
      <template v-slot:apyCell="pool">
        <div class="px-6 py-4 -mt-1 flex justify-end">
          {{
            Number(pool.dynamic.apy.pool) > 10000
              ? '-'
              : fNum(pool.dynamic.apy.total, 'percent')
          }}
          <LiquidityMiningTooltip :pool="pool" />
        </div>
      </template>
    </BalTable>
  </BalCard>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import { useRouter } from 'vue-router';
import { useI18n } from 'vue-i18n';

import { DecoratedPoolWithShares } from '@/services/balancer/subgraph/types';

import { getAddress } from '@ethersproject/address';

import useNumbers from '@/composables/useNumbers';
import useTokens from '@/composables/useTokens';
import useFathom from '@/composables/useFathom';
import useAccountBalances from '@/composables/useAccountBalances';

import LiquidityMiningTooltip from '@/components/tooltips/LiquidityMiningTooltip.vue';

import { ColumnDefinition } from '../_global/BalTable/BalTable.vue';

export default defineComponent({
  components: {
    LiquidityMiningTooltip
  },

  emits: ['loadMore'],

  props: {
    data: {
      type: Array
    },
    isLoading: {
      type: Boolean
    },
    isLoadingMore: {
      type: Boolean,
      default: false
    },
    showPoolShares: {
      type: Boolean,
      default: false
    },
    noPoolsLabel: {
      type: String,
      default: 'No pools'
    },
    isPaginated: {
      type: Boolean,
      default: false
    }
  },

  setup(props) {
    const { fNum } = useNumbers();
    const { allTokens } = useTokens();
    const router = useRouter();
    const { t } = useI18n();
    const { trackGoal, Goals } = useFathom();
    const {
      balances,
      hasBalance,
      isLoading: isLoadingBalances,
      isIdle: isBalancesQueryIdle
    } = useAccountBalances();

    // COMPOSABLES
    const columns = ref<ColumnDefinition<DecoratedPoolWithShares>[]>([
      {
        name: 'Icons',
        id: 'icons',
        accessor: 'uri',
        Header: 'iconColumnHeader',
        Cell: 'iconColumnCell',
        width: 125,
        noGrow: true
      },
      {
        name: t('composition'),
        id: 'poolName',
        accessor: 'id',
        Cell: 'poolNameCell',
        width: 350
      },
      {
        name: t('myBalance'),
        accessor: pool => fNum(pool.shares, 'usd', { forcePreset: true }),
        align: 'right',
        id: 'myBalance',
        hidden: !props.showPoolShares,
        sortKey: pool => Number(pool.shares),
        width: 150
      },
      {
        name: t('poolValue'),
        accessor: pool => fNum(pool.totalLiquidity, 'usd'),
        align: 'right',
        id: 'poolValue',
        sortKey: pool => Number(pool.totalLiquidity),
        width: 150
      },
      {
        name: t('volume24h', [t('hourAbbrev')]),
        accessor: pool => fNum(pool.dynamic.volume, 'usd'),
        align: 'right',
        id: 'poolVolume',
        sortKey: pool => Number(pool.dynamic.volume),
        width: 150
      },
      {
        name: t('apy'),
        Cell: 'apyCell',
        accessor: pool => pool.dynamic.apy.total,
        align: 'right',
        id: 'poolApy',
        sortKey: pool => Number(pool.dynamic.apy.total),
        width: 150
      }
    ]);

    function sortedTokenAddressesFor(pool: DecoratedPoolWithShares) {
      const sortedTokens = sortedTokensFor(pool);
      return sortedTokens.map(token => getAddress(token.address));
    }

    function sortedTokensFor(pool: DecoratedPoolWithShares) {
      const sortedTokens = pool.tokens.slice();
      sortedTokens.sort((a, b) => parseFloat(b.weight) - parseFloat(a.weight));
      return sortedTokens;
    }

    function handleRowClick(pool: DecoratedPoolWithShares) {
      trackGoal(Goals.ClickPoolsTableRow);
      router.push({ name: 'pool', params: { id: pool.id } });
    }

    return {
      // data
      columns,
      allTokens,
      balances,
      isLoadingBalances,
      isBalancesQueryIdle,

      // methods
      handleRowClick,
      getAddress,
      fNum,
      sortedTokenAddressesFor,
      sortedTokensFor,
      hasBalance
    };
  }
});
</script>
