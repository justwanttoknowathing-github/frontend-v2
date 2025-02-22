<template>
  <div>
    <div class="flex items-center flex-wrap">
      <div class="flex items-center flex-wrap">
        <BalBtn
          color="gray"
          outline
          size="sm"
          @click="onClick"
          class="mb-2 md:mb-0 mr-4"
        >
          <BalIcon name="search" size="sm" class="mr-2" />
          {{ $t('filterByToken') }}
        </BalBtn>
        <BalChip
          v-for="token in modelValue"
          class="mr-2"
          :key="token"
          color="white"
          iconSize="sm"
          :closeable="true"
          @closed="removeToken(token)"
        >
          <BalAsset :address="token" :size="20" class="flex-auto" />
          <span class="ml-2">{{ tokenDictionary[token]?.symbol }}</span>
        </BalChip>
      </div>
      <div
        v-if="
          account &&
            !isNotFetchingBalances &&
            !isLoadingBalances &&
            !hasNoBalances
        "
        class="text-gray-400 overflow-x-auto"
      >
        <span class="mr-6">{{ $t('inYourWallet') }}</span>
        <span
          v-for="token in sortedBalances"
          :key="`wallet-${token.symbol}`"
          class="mr-6 cursor-pointer hover:text-blue-700"
          @click="addToken(token.address)"
        >
          {{ token?.symbol }}
        </span>
      </div>
      <div v-else class="text-gray-400 flex flex-wrap">
        <span class="mr-6">{{ $t('popularBases') }}</span>
        <span
          v-for="token in whiteListedTokens"
          :key="`popular-${token.symbol}`"
          class="mr-6 cursor-pointer hover:text-gray-700"
          @click="addToken(token.address)"
        >
          {{ token?.symbol }}
        </span>
      </div>
    </div>
    <teleport to="#modal">
      <SelectTokenModal
        v-if="selectTokenModal"
        :excluded-tokens="modelValue"
        @close="selectTokenModal = false"
        @select="addToken"
      />
    </teleport>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, PropType, ref } from 'vue';
import SelectTokenModal from '@/components/modals/SelectTokenModal/SelectTokenModal.vue';
import useAccountBalances from '@/composables/useAccountBalances';
import { sortBy, take } from 'lodash';
import useWeb3 from '@/composables/useWeb3';
import { TOKENS } from '@/constants/tokens';
import useTokenLists from '@/composables/useTokenLists';
import { ETHER } from '@/constants/tokenlists';
import { getAddress } from '@ethersproject/address';

export default defineComponent({
  name: 'TokenSearchInput',

  components: {
    SelectTokenModal
  },

  emits: ['update:modelValue'],

  props: {
    modelValue: { type: Array as PropType<string[]>, default: () => [] },
    loading: { type: Boolean, default: true }
  },

  setup(props, { emit }) {
    // COMPOSABLES
    const { tokenDictionary } = useTokenLists();
    const {
      isLoading: isLoadingBalances,
      balances,
      isIdle: isNotFetchingBalances
    } = useAccountBalances();
    const { account } = useWeb3();

    // sorted by biggest bag balance, limited to biggest 5
    const sortedBalances = computed(() => {
      return take(
        sortBy(Object.values(balances.value || {}), 'balance')
          .reverse()
          .filter(
            (balance: any) =>
              !props.modelValue.includes(balance.address) &&
              balance.address !== ETHER.address
          ),
        6
      );
    });

    const hasNoBalances = computed(() => !sortedBalances.value.length);
    const whiteListedTokens = computed(() =>
      Object.values(tokenDictionary.value)
        .filter(token => TOKENS.Popular.Symbols.includes(token.symbol))
        .filter(balance => !props.modelValue.includes(balance.address))
    );

    // DATA
    const selectTokenModal = ref(false);

    // METHODS
    function addToken(token: string) {
      let _token = token;
      // special case for ETH where we want it to filter as WETH regardless
      // as ETH is the native asset
      if (getAddress(token) === ETHER.address) {
        _token = TOKENS.AddressMap.WETH;
      }
      const newSelected = [...props.modelValue, _token];
      emit('update:modelValue', newSelected);
    }

    function removeToken(token: string) {
      const newSelected = props.modelValue.filter(t => t !== token);
      emit('update:modelValue', newSelected);
    }

    function onClick() {
      if (!props.loading) selectTokenModal.value = true;
    }

    return {
      // data
      selectTokenModal,
      isNotFetchingBalances,
      isLoadingBalances,
      balances,
      sortedBalances,
      account,
      hasNoBalances,
      whiteListedTokens,
      // computed
      tokenDictionary,
      // methods
      addToken,
      removeToken,
      onClick
    };
  }
});
</script>
