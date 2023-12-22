<template>
    <div class="conatiner-fluid d-flex flex-column align-items-center m-5">
        <h3 class="text-center">There is an error in fetching the data!</h3>
        <p class="my-4 text-center">Error message: {{ errorMessage }}</p>
        <router-link to="/" class="btn btn-light">Back to homepage</router-link>
    </div>
</template>

<script>
export default {
  data() {
    return {
      errorMessage: '',
    };
  },
  created() {
    this.errorMessage = this.$route.query.message || 'An error has occured.';
  },

};

</script>
