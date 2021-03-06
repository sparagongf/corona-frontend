<template>
  <div class="flex flex-wrap">
    <SidebarNav></SidebarNav>

    <div class="w-full lg:w-5/6 px-2 pt-2 bg-gray-200">
      <div class="pl-2">
        <p class="text-2xl font-bold">COVID-19 {{ $t('Overview') }}</p>

        <p class="text-xs font-bold leading-tight my-2 text-gray-600">
          {{ $t('Sources') }}: WHO, CDC, ECDC, NHC of the PRC, JHU CSSE, DXY, QQ, {{ $t('and various international media') }}
        </p>
      </div>

      <div class="flex flex-wrap">
        <div class="w-full lg:w-1/2 px-2">
          <div class="max-w-full rounded shadow-md bg-white p-3 mb-5">
            <div class="flex flex-row lg:flex-row">
              <div class="px-2">
                <p class="text-sm font-bold text-red-600">{{ $t('Total Confirmed') }}</p>
                <p class="text-xl font-bold text-red-600">{{ confirmed | formatNumber }}</p>
              </div>

              <div class="px-2">
                <p class="text-sm font-bold text-green-600">{{ $t('Total Recovered') }}</p>
                <p class="text-xl font-bold text-green-600">{{ recovered | formatNumber }}</p>
              </div>

              <div class="px-2">
                <p class="text-sm font-bold text-gray-600">{{ $t('Total Deaths') }}</p>
                <p class="text-xl font-bold text-gray-600">{{ deaths | formatNumber }}</p>
              </div>
            </div>
          </div>

          <div class="w-full rounded shadow-md bg-white p-3 mb-5 block lg:hidden">
            <div class="mt-3" style="max-height: 36.3rem; overflow: auto;">
              <table class="table-auto w-full">
                <thead class="text-xs leading-tight border-b-2">
                <tr>
                  <th class="border px-2 py-2">{{ $t('Country') }}</th>
                  <th class="border px-1 py-2">{{ $t('Total Confirmed') }}</th>
                  <th class="border px-1 py-2">{{ $t('Total Recovered') }}</th>
                  <th class="border px-1 py-2">{{ $t('Total Deaths') }}</th>
                </tr>
                </thead>
                <tbody class="font-bold">
                <tr v-for="loc in affectedCountries" :key="loc.countryName">
                  <td class="bg-gray-200 text-xs border px-2 py-2 hover:bg-primary hover:text-white">
                    <template v-if="loc.countryName === 'Others'">
                      <span>{{loc.countryName}}</span>
                      <a href="#notes-on-others">*</a>
                    </template>
                    <template v-else-if="loc.countryCode">
                      <nuxt-link :to="`/country/${loc.countryCode.toLowerCase()}`" style="display: block;">
                        <Flag :country-code="loc.countryCode"></Flag>
                        {{loc.countryName}}
                      </nuxt-link>
                    </template>
                  </td>
                  <td class="text-center border px-1 py-2">{{ loc.confirmed | formatNumber }}</td>
                  <td class="text-center border px-1 py-2">{{ loc.recovered | formatNumber }}</td>
                  <td class="text-center border px-1 py-2">{{ loc.deaths | formatNumber }}</td>
                </tr>
                </tbody>
              </table>
              <div class="my-2 font-bold text-xs text-gray-600 leading-tight">
                * {{ $t('Cases identified on a cruise ship currently in Japanese territorial waters.') }}
                <a name="notes-on-others" class="anchor"></a>
              </div>
            </div>
          </div>

          <div class="max-w-full rounded shadow-md bg-white p-3 mb-5">
            <OutbreakTrendChart :data="outbreakTrendData"></OutbreakTrendChart>
          </div>

          <div class="max-w-full rounded shadow-md bg-white p-3 mb-5">
            <TopCountryWithDailyNewCases :data="topCountryWithDailyNewCasesData"/>
          </div>
        </div>

        <div class="w-full lg:w-1/2 px-2">
          <div class="max-w-full rounded shadow-md bg-white p-3 mb-5">
            <AffectedCountry :data="affectedCountryData"></AffectedCountry>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SidebarNav from '~/components/Analytics/SidebarNav'
import OutbreakTrendChart from '~/components/Analytics/OutbreakTrend'
import TopCountryWithDailyNewCases from '~/components/Analytics/TopCountryWithDailyNewCases'
import AffectedCountry from '~/components/Analytics/AffectedCountry'
import Flag from '~/components/Flag';

export default {
  head() {
    const title = this.$t('COVID-19 related analytics, graphs, and charts');
    const description = this.$t('Visual presentations of COVID-19 related data from verified sources such as WHO, CDC, ECDC, NHC of the PRC, JHU CSSE, DXY, QQ, and various international media.');

    return {
      title,
      meta: [
        { hid: 'title', name: 'title', content: title },
        { hid: 'description', name: 'description', content: description },
        { hid: 'og-title', property: 'og:title', content: title },
        { hid: 'og-description', property: 'og:title', content: description },
        { hid: 'twitter-title', property: 'twitter:title', content: title },
        { hid: 'twitter-description', property: 'twitter:title', content: description },
      ],
    };
  },

  components: { SidebarNav, OutbreakTrendChart, TopCountryWithDailyNewCases, AffectedCountry, Flag },

  mounted () {
    this.loadStats()
    this.loadOutbreakTrend()
    this.loadTopCountryWithDailyNewCases()
    this.loadAffectedCountry()
  },

  data () {
    return {
      currentDate: new Date,
      confirmed: 0,
      deaths: 0,
      recovered: 0,
      outbreakTrendData: [],
      topCountryWithDailyNewCasesData: [],
      affectedCountryData: [],
    }
  },

  computed: {
    affectedCountries() {
      return this.affectedCountryData.filter(i => i.confirmed && i.countryCode)
    }
  },

  methods: {
    loadStats () {
      this.$api.stats.getGlobalStats()
        .then(data => {
          this.confirmed = data.totalConfirmed
          this.deaths = data.totalDeaths
          this.recovered = data.totalRecovered
        })
    },

    loadOutbreakTrend () {
      this.$api.analytics.fetchTrendByDate('2020-01-27', this.currentDate.toISOString().slice(0, 10))
        .then(data => {
          this.outbreakTrendData = data
        })
    },

    loadTopCountryWithDailyNewCases () {
      this.$api.analytics.fetchTopCountryWithDailyNewCases()
        .then(data => {
          this.topCountryWithDailyNewCasesData = data
        })
    },

    loadAffectedCountry () {
      this.$api.analytics.fetchAllAffectedCountry()
        .then(data => {
          this.affectedCountryData = data
        })
    },
  }
}
</script>
