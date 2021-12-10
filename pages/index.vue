<template lang="pug">
  v-container
    v-row
      v-col(cols)
        v-card( 
          flat
          tile
          style="max-width: 700px"
        ).d-flex.flex-row.justify-center.align-center.mb-6
          v-text-field( 
              v-model="keyword"
              :counter="10"
              label="Search..."
          ).mr-4
          v-select( 
              v-model="gender"
              :items="genderTypes"
              label="Gender"
              style="width: 200px"
          ).mr-4
          v-select( 
              v-model="year"
              :items="years"
              label="Year"
              style="width: 200px"
          ).mr-4
          v-select( 
              v-model="country"
              :items="countries"
              label="Country"
              style="width: 200px"
          ).mr-4
          v-btn(
            v-if="showClearFilters"
            color="secondary"
            @click="clearFilters"
          ) Clear Filters
        DataTable(
          v-if="itemsData.length"
          :headers="headers"
          :items="itemsData"
        )
        v-progress-circular(
          v-if="!itemsData.length && hasResults"
          width="2"
          color="rs__primary"
          indeterminate
        ).mx-auto 
        div(
          v-if="!itemsData.length && !hasResults"
        ).no-data No data to display, try broadening your filters...
        
</template>

<script>
import DataTable from '~/components/DataTable.vue'
import sales from '~/api/sales.js'
import {debounce} from '~/helpers/helpers.js'

export default {
  name: 'Index',
  components: {
    DataTable
  },
  data() {
    return {
      sales,
      items: [],
      itemsData: [],
      initialLoad: true,
      years: [],
      year: null,
      keyword: '',
      gender: '',
      genderTypes: [],
      country: '',
      countries: [],
      headers: [
        { text: 'Name', value: 'user.first_name', align: 'start' },
        { text: 'Last Name', value: 'user.last_name', align: 'start' },
        { text: 'Email', value: 'email' },
        { text: 'Gender', value: 'gender' },
        { text: 'Year', value: 'year' },
        { text: 'Sales', value: 'sales' },
        { text: 'Country', value: 'country' },
      ],
    }
  },
  watch: {
    keyword() {
      return this.filterItems();
    },
    year() {
      if(this.year) {
        return this.filterItems();
      }
    },
    gender() {
      if(this.gender) {
        return this.filterItems();
      }
    },
    country() {
      if(this.country) {
        return this.filterItems();
      }
    },
  },
  async created() {
    this.items = await this.fetchData(0, 50)
    this.itemsData = this.items;
    
    this.mapData();
  },
  methods: {
    mapData() {
      this.generateCountriesArray();
      this.generateGenderTypes();
      this.generateYearsArray();
    },
    async fetchData(page, size) {
      const start = page * size
      await this.delay(3000)
      this.initialLoad = false
      return await sales.results.slice(start, start + size)
    },
    generateYearsArray() {
      let yearsArray = this.items.map(item => item.year);
      this.years = [...new Set(yearsArray.sort())];
    },
    generateCountriesArray() {
      let countriesList = this.items.map(item => item.country);
      this.countries = [...new Set(countriesList.sort())];
    },
    generateGenderTypes() {
      let typesOfGender = this.items.map(item => item.gender);
      this.genderTypes = [...new Set(typesOfGender)];
    },
    clearFilters() {
      this.year = null;
      this.keyword = '';
      this.gender = '';
      this.country = '';
      this.filterItems();
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },
    filterItems: debounce(function () {
      this.itemsData = this.items.filter(
          item => 
                  (this.keyword ? (
                    item.user.first_name.toLowerCase().includes(this.keyword.toLowerCase()) ||
                    item.user.last_name.toLowerCase().includes(this.keyword.toLowerCase()) || 
                    item.email.toLowerCase().includes(this.keyword.toLowerCase()) ||
                    item.sales.includes(this.keyword)
                  ) : true) &&
                  (this.year ? item.year === this.year : true) &&
                  (this.gender ? item.gender === this.gender : true) &&
                  (this.country ? item.country === this.country : true) 
      )
    }, 500),
  },
  computed: {
    hasResults() {
      return this.initialLoad ? true : this.itemsData.length;
    },
    showClearFilters() {
      return  this.year || this.keyword || this.gender || this.country;
    }
  }
}
</script>

<style lang="sass" scoped>
.v-progress-circular
  position: absolute
  top: 50%
  left: 50%

.no-data
  position: absolute
  top: 50%
  left: 50%
  transform: translate(-50%, -50%)
  font-size: 1.2rem
</style>