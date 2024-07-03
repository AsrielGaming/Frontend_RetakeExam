<template>
  <div>
    <!-- Main container -->
    <div class="main-container">
      
      <!-- Banner container -->
      <div class="banner">
        <h3>All Booking</h3>
      </div>

      <!-- Content container -->
      <div class="content">
        <!-- Booking container -->
        <div class="booking-container">
          <!-- Placeholder for Booking -->
          <div>
            <p>No available booking.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'BookingPage',
  data() {
    return {
      // Any data specific to Booking can be added here
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

/* Styling for booking container */
.booking-container {
  width: 100%;
}

/* Styling for individual booking container */
.booking-spot-container {
  border: 1px solid #ccc;
  margin-bottom: 10px;
  padding: 10px;
  display: flex;
}
</style>
