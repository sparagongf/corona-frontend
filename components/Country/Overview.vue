<template>
  <div class="max-w-full rounded border border-gray-400 bg-white p-3 h-full">
    <p class="text-xl font-bold pl-2">
      <Flag :country-code="country.code"></Flag>
      {{country.name}} {{ $t('Overview') }}
      </p>
    <div class="flex flex-row lg:flex-row pt-3 justify-center sm:justify-start md:justify-start lg:justify-start">
      <div class="px-2 text-center">
        <p class="text-2xl font-bold text-red-600">{{ info.confirmed | formatNumber }}</p>
        <p class="text-l font-bold text-gray-600">{{ $t('Confirmed') }}</p>
        <p class="text-xs font-bold text-red-600">{{ $t('+{number} since yesterday', { number: $options.filters.formatNumber(info.diffConfirmed) }) }}</p>
      </div>

      <div class="px-2 text-center">
        <p class="text-2xl font-bold text-green-600">{{ info.recovered | formatNumber }}</p>
        <p class="text-l font-bold text-gray-600">{{ $t('Recovered') }}</p>
        <!-- <p class="text-xs font-bold text-green-600">{{ $t('+{number} since yesterday', { number: $options.filters.formatNumber(info.diffRecovered) }) }}</p> -->
      </div>

      <div class="px-2 text-center">
        <p class="text-2xl font-bold text-gray-600">{{ info.deaths | formatNumber }}</p>
        <p class="text-l font-bold text-gray-600">{{ $t('Deaths') }}</p>
        <p class="text-xs font-bold text-gray-600">{{ $t('+{number} since yesterday', { number: $options.filters.formatNumber(info.diffDeaths) }) }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import Flag from "~/components/Flag"
export default {
  props: {
    info: {
      type: Object,
      required: true,
    },
    country: {
      type: Object,
      required: true
    }
  },
  components: {
    Flag
  }
}
</script>

<style scoped>
</style>
