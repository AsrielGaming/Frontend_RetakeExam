<template>
  <div>
    <!-- Main container -->
    <div class="main-container">
      
      <!-- Banner container -->
      <div class="banner">
        <h3>All Listings</h3>
      </div>

      <!-- Content container -->
      <div class="content">
        <!-- Listing container -->
        <div class="listing-container">
          <!-- Placeholder for Listings -->
          <div>
            <p>No available listings.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ListingPage',
  props: {
    // Userdata property
    userData: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      // Any data specific to Listings can be added here
    };
  }
};
</script>

<style scoped>
/* Styling for main container */
.main-container {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

/* Styling for banner container */
.banner {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
}

/* Styling for content container */
.content {
  display: flex;
  border: 1px solid #ccc;
  padding: 10px;
}

/* Styling for listing container */
.listing-container {
  width: 100%;
}

/* Styling for individual listing container */
.listing-spot-container {
  border: 1px solid #ccc;
  margin-bottom: 10px;
  padding: 10px;
  display: flex;
}
</style>
