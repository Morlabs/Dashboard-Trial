<template>
  <div class="private-pool-view">
    <info-bar
      status="private"
      :title="$t(`home-page.private-pool-view.info-bar.title--${poolId}`)"
      :subtitle="$t(`home-page.private-pool-view.info-bar.subtitle--${poolId}`)"
      :description="
        $t(`home-page.private-pool-view.info-bar.description--${poolId}`)
      "
      :indicators="barIndicators"
      :is-loading="isInitializing"
    />
    <info-dashboard
      :indicators="dashboardIndicators"
      :progress="dashboardProgress"
      :is-loading="isInitializing || isUserDataUpdating"
    >
      <app-button
        class="private-pool-view__dashboard-btn"
        :text="$t('home-page.private-pool-view.claim-btn')"
        :is-loading="isInitializing"
        :disabled="isClaimDisabled"
        @click="isClaimModalShown = true"
      />
      <claim-modal
        v-if="!isClaimDisabled && currentUserReward"
        v-model:is-shown="isClaimModalShown"
        :amount="formatEther(currentUserReward)"
        :pool-id="poolId"
      />
    </info-dashboard>
  </div>
</template>

<script lang="ts" setup>
import { AppButton, ClaimModal, InfoBar, InfoDashboard } from '@/common'
import { useI18n, usePool } from '@/composables'
import { DEFAULT_TIME_FORMAT } from '@/const'
import { ICON_NAMES } from '@/enums'
import type { InfoBarType, InfoDashboardType, ProgressBarType } from '@/types'
import { formatEther, BigNumber, Time } from '@/utils'
import { computed, ref } from 'vue'

const props = defineProps<{ poolId: number }>()

const isClaimModalShown = ref(false)

const poolId = computed(() => props.poolId)

const { t } = useI18n()

const {
  currentUserReward,
  dailyReward,
  poolData,
  userPoolData,

  isClaimDisabled,

  isInitializing,
  isUserDataUpdating,
} = usePool(poolId.value)

const barIndicators = computed<InfoBarType.Indicator[]>(() => [
  {
    title: t('home-page.private-pool-view.daily-reward-title'),
    value: dailyReward.value ? `${formatEther(dailyReward.value)} AGEN` : '',
  },
  {
    title: t('home-page.private-pool-view.started-at-title'),
    value: poolData.value
      ? new Time(poolData.value.payoutStart.toNumber()).format(
          DEFAULT_TIME_FORMAT,
        )
      : '',
  },
  {
    title: t('home-page.private-pool-view.claim-at-title'),
    value: poolData.value
      ? new Time(
          poolData.value.payoutStart
            .add(poolData.value.claimLockPeriod)
            .toNumber(),
        ).format(DEFAULT_TIME_FORMAT)
      : '',
    note: t('home-page.private-pool-view.claim-at-note'),
  },
])

const dashboardIndicators = computed<InfoDashboardType.Indicator[]>(() => [
  {
    iconName: ICON_NAMES.arbitrum,
    title: t('home-page.private-pool-view.available-to-claim-title'),
    value: currentUserReward.value
      ? `${formatEther(currentUserReward.value)} AGEN`
      : '',
  },
])

const dashboardProgress = computed<ProgressBarType.Progress>(() => ({
  value: userPoolData.value?.deposited || BigNumber.from('0'),
  total: poolData.value?.totalDeposited || BigNumber.from('1'),
}))
</script>

<style lang="scss" scoped>
.private-pool-view .private-pool-view__dashboard-btn {
  min-width: toRem(184);

  @include respond-to(medium) {
    width: 100%;
  }
}
</style>
