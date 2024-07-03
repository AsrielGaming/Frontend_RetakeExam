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
        <div class="filter-controls">
          <button>All</button>
          <button>Available</button>
          <button>Comment</button>
        </div>

        <!-- Additional dropdowns -->
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
        
        <div class="dropdown">
          <select>
            <option value="any">Any Size</option>
            <option value="small">Small (≤ 50 m²)</option>
            <option value="medium">Medium (51-100 m²)</option>
            <option value="large">Large (> 100 m²)</option>
          </select>
        </div>
        
        <div class="dropdown">
          <select>
            <option value="any">Any Price</option>
            <option value="cheap">Cheap (≤ 50 €)</option>
            <option value="normal">Normal (51-100 €)</option>
            <option value="luxury">Luxury (> 100 €)</option>
          </select>
        </div>

        <!-- Amenity dropdown (multi-select) using vue-multiselect -->
        <div class="dropdown multiselect-dropdown">
          <multiselect v-model="selectedAmenities"
                       :options="amenitiesList"
                       placeholder="Select amenities"
                       :multiple="true"
                       :close-on-select="false"
                       :clear-on-select="false"
                       :preserve-search="true"
                       :preselect-first="false">
            <template slot="selection" slot-scope="{ values, isOpen }">
              <span class="multiselect__single" v-if="values.length && !isOpen">{{ values.length }} options selected</span>
            </template>
          </multiselect>
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
import axios from 'axios';
import Multiselect from 'vue-multiselect'; // Import vue-multiselect component

export default {
  name: 'CampingSpotPage',
  components: {
    Multiselect // Register Multiselect component
  },
  data() {
    return {
      amenitiesList: [], // Array to store amenities fetched from API
      selectedAmenities: [] // Array to store selected amenities for filtering
    };
  },
  mounted() {
    this.fetchAmenities(); // Fetch amenities when the component is mounted
  },
  methods: {
    async fetchAmenities() {
      try {
        // Fetch amenities from your API
        const response = await axios.get('http://localhost:5235/Amenity');
        const data = response.data.map(amenity => amenity.name); // Extract amenities names
        this.amenitiesList = data; // Populate the amenities list

      } catch (error) {
        console.error('Error fetching amenities:', error);
      }
    }
  }
};
</script>

<!-- Add Multiselect CSS. Can be added as a static asset or inside a component. -->
<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

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
  align-items: center; /* Center items vertically */
  margin-bottom: 10px;
  border: 1px solid #ccc;
  padding: 10px;
  position: relative; /* Ensure positioned relatively for absolute children */
}

.filter-controls {
  display: flex;
  align-items: center;
}

.filter-controls > *:not(:last-child) {
  margin-right: 5px; /* Adjust margin between elements */
}

.filter-container button {
  text-align: center;
  margin: 0 5px; /* Adjust margin for spacing between buttons */
}

/* Styling for dropdown */
.dropdown {
  position: relative; /* Ensure positioned relatively for absolute children */
  margin: 0 5px; /* Adjust margin for spacing between dropdowns */
}

.dropdown select {
  width: 200px; /* Adjust as needed */
  padding: 5px;
}

/* Styling for multiselect dropdown */
.multiselect-dropdown .multiselect__content-wrapper {
  position: absolute;
  top: 100%;
  left: 0;
  z-index: 1000; /* Ensure it's above other content */
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
