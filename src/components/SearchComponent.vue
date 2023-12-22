<template>
  <div class="wrapper position-relative d-flex flex-column align-items-center mt-5">
    <div class="search-box p-3 bg-light rounded-2 shadow-sm z-1">
      <label for="searchInput" class="w-100"
        ><input
          type="text"
          id="searchInput"
          class="form-control border-0 shadow-sm p-3 text-secondary fs-2 text-center
          w-100 shadow-none"
          placeholder="City or State"
          v-model="searchQuery"
      /></label>
    </div>

    <p class="m-0 pt-3 text-center text-danger" v-if="noDataMessage">No data available</p>

    <div v-if="filteredCities.length !== 0" class="display-list m-0 position-absolute">
      <ul class="suggestions list-unstyled">
        <li
          class="border-bottom p-3 text-secondary d-flex
          justify-content-between align-items-cente shadow-sm"
          v-for="city in filteredCities"
          :key="city"
        >
          <div class="fs-5">
            <span>
              {{ city.city.substring(0, city.cityStartIndex) }}
            </span>
            <span class="highlight">
              {{ city.city.substring(city.cityStartIndex, city.cityEndIndex) }}
            </span>
            <span> {{ city.city.substring(city.cityEndIndex) }}, </span>
            <span>
              {{ city.state.substring(0, city.stateStartIndex) }}
            </span>
            <span class="highlight">
              {{ city.state.substring(city.stateStartIndex, city.stateEndIndex) }}
            </span>
            <span> {{ city.state.substring(city.stateEndIndex) }} </span>
          </div>
          <span class="fs-6 ms-2">{{ formatNumber(city.population) }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SearchComponent',

  data() {
    return {
      cities: [],
      searchQuery: '',
      filteredCities: [],
      noDataMessage: false,
    };
  },
  watch: {
    searchQuery() {
      this.checkQuery();
    },
  },

  methods: {
    checkQuery() {
      if (this.searchQuery.trim() === '') {
        this.filteredCities = [];
        this.noDataMessage = false;
      } else {
        this.requestData();
      }
    },

    requestData() {
      const url = 'https://gist.githubusercontent.com/Miserlou/c5cd8364bf9b2420bb29/raw/2bf258763cdddd704f8ffd3ea9a3e81d25e2c6f6/cities.json';
      this.axios
        .get(url)
        .then((response) => {
          if (response.status === 200 && response.data.length !== 0) {
            this.cities = response.data;
            this.filterData();
          } else if (response.data.length === 0) {
            this.cities = [];
          }
        })
        .catch((error) => {
          console.error('Error fetching data:', error.message);
          this.$router.push({ name: 'error', query: { message: error.message } });
        });
    },

    filterData() {
      if (this.cities.length !== 0) {
        const query = this.searchQuery.toLowerCase();
        this.filteredCities = this.cities.filter((item) => {
          const regex = new RegExp(query, 'gi');
          return item.city.match(regex) || item.state.match(regex);
        });
        this.highlight();
      } else {
        this.noDataMessage = true;
      }
    },

    highlight() {
      this.filteredCities.forEach((item) => {
        item.cityStartIndex = -1;
        item.cityEndIndex = -1;
        item.stateStartIndex = -1;
        item.stateEndIndex = -1;

        const query = this.searchQuery.toLowerCase();

        const cityIndex = item.city.toLowerCase().indexOf(query);
        if (cityIndex !== -1) {
          item.cityStartIndex = cityIndex;
          item.cityEndIndex = cityIndex + query.length;
        }

        const stateIndex = item.state.toLowerCase().indexOf(query);
        if (stateIndex !== -1) {
          item.stateStartIndex = stateIndex;
          item.stateEndIndex = stateIndex + query.length;
        }
      });
    },

    formatNumber(population) {
      return population.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    },
  },
};
</script>
