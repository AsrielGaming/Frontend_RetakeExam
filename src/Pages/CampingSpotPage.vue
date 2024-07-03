<template>
  <div>
    <!-- Main container -->
    <div class="main-container">
      
      <!-- Banner container -->
      <div class="banner">
        <h3>All camping spots</h3>
      </div>

      <!-- Filter container -->
      <div class="filter-container">
        <button>All</button>
        <button>Available</button>
        <button>Amenities</button>
        <button>Comment</button>

        <!-- Rating dropdown -->
        <div class="dropdown">
          <select>
            <option value="any">Any Rating</option>
            <option value="1star">1 star rating</option>
            <option value="2star">2 star rating</option>
            <option value="3star">3 star rating</option>
            <option value="4star">4 star rating</option>
            <option value="5star">5 star rating</option>
          </select>
        </div>
        
        <!-- Size dropdown -->
        <div class="dropdown">
          <select>
            <option value="any">Any Size</option>
            <option value="small">Small (≤ 50 m²)</option>
            <option value="medium">Medium (51-100 m²)</option>
            <option value="large">Large (> 100 m²)</option>
          </select>
        </div>
        
        <!-- Price dropdown -->
        <div class="dropdown">
          <select>
            <option value="any">Any Price</option>
            <option value="cheap">Cheap (≤ 50 €)</option>
            <option value="normal">Normal (51-100 €)</option>
            <option value="luxury">Luxury (> 100 €)</option>
          </select>
        </div>
      </div>
      
      <!-- Content container -->
      <div class="content">
        <!-- Campingspots container -->
        <div class="campingspots-container">
          <!-- Placeholder for camping spots -->
          <div>
            <p>No available camping spots.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CampingSpotPage',
  // Add component logic here if needed
}
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

/* Styling for filter container */
.filter-container {
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  padding: 10px;
}

.filter-container button {
  margin-right: 10px;
  text-align: center;
}

/* Styling for dropdown */
.dropdown {
  margin-right: 10px;
}

.dropdown select {
  padding: 5px;
}

/* Styling for content container */
.content {
  display: flex;
  border: 1px solid #ccc;
  padding: 10px;
}

/* Styling for campingspots container */
.campingspots-container {
  width: 100%;
}

/* Styling for individual camping spot container */
.camping-spot-container {
  border: 1px solid #ccc;
  margin-bottom: 10px;
  padding: 10px;
  display: flex;
}

</style>
