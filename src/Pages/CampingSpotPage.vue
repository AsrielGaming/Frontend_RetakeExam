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
                       :options="amenitiesList.map(amenity => amenity.name)"
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
          <div v-if="isLoading">
            <p>Loading camping spots...</p>
          </div>
          <div v-else-if="campingSpots.length">
            <div v-for="spot in campingSpots" :key="spot.id" class="camping-spot-container">
              <!-- Left side for spot details -->
              <div class="spot-details">
                <div class="camping-spot">
                  <h3>{{ spot.spotName }}</h3>
                  <p>Description: {{ spot.description }}</p>
                  <!--<p>CampType IDs: {{ spot.campTypes }}</p>-->
                  <p>CampTypes: {{ getCampTypeNames(spot.campTypes) }}</p> <!-- Display CampType names here -->
                  <p>Size: {{ spot.size }} m²</p>
                  <p>Price: {{ spot.price }} €</p>
                  <p>Availability: {{ spot.isAvailable ? 'Available' : 'Not Available' }}</p>
                  <p>Camping Ground: {{ getCampingGroundName(spot.campingGroundId) }}</p>
                  <p>Owner: {{ getUserName(spot.userId) }}</p>
                </div>
              </div>

              <!-- Middle section for booking -->
              <div class="booking-section">
                <h3>Book this spot</h3>
                <div class="date-picker-section">
                  <label>Date:</label>
                  <input type="date" v-model="spot.startingDate">
                </div>
                <div class="time-inputs">
                  <div class="time-picker-group">
                    <label for="start-time">Start Time:</label>
                    <input type="time" id="start-time" v-model="spot.startTime">
                  </div>
                  <div class="time-picker-group">
                    <label for="end-time">End Time:</label>
                    <input type="time" id="end-time" v-model="spot.endTime">
                  </div>
                </div>
                <div class="book-button-container">
                  <button>Book</button>
                </div>
              </div>

              <!-- Right side for amenities -->
              <div class="amenities-section">
                <h3>Amenities</h3>
                <ul>
                  <li v-for="amenityId in spot.amenities" :key="amenityId">
                    {{ getAmenityName(amenityId) }}
                  </li> <!-- Display amenity names -->
                </ul>
              </div>
            </div>
          </div>
          <div v-else>
            <p>No available camping spots.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Multiselect from 'vue-multiselect';

export default {
  name: 'CampingSpotPage',
  components: {
    Multiselect
  },
  data() {
    return {
      amenitiesList: [],
      selectedAmenities: [],
      campingGrounds: [],
      campingSpots: [],
      users: [],
      campTypes: [], // Array to store camp types fetched from API
      isLoading: true
    };
  },
  mounted() {
    this.initializeData();
  },
  methods: {
    async initializeData() {
      try {
        await Promise.all([
          this.fetchAmenities(),
          this.fetchCampingGrounds(),
          this.fetchUsers(),
          this.fetchCampingSpots(),
          this.fetchCampTypes()
        ]);
      } catch (error) {
        console.error('Error initializing data:', error);
      } finally {
        this.isLoading = false;
      }
    },
    async fetchAmenities() {
      try {
        const response = await axios.get('http://localhost:5235/Amenity');
        this.amenitiesList = response.data;
      } catch (error) {
        console.error('Error fetching amenities:', error);
      }
    },
    async fetchCampingGrounds() {
      try {
        const response = await axios.get('http://localhost:5235/CampingGround');
        this.campingGrounds = response.data;
      } catch (error) {
        console.error('Error fetching camping grounds:', error);
      }
    },
    async fetchCampingSpots() {
      try {
        const response = await axios.get('http://localhost:5235/CampingSpot');
        this.campingSpots = response.data;
      } catch (error) {
        console.error('Error fetching camping spots:', error);
      }
    },
    async fetchUsers() {
      try {
        const response = await axios.get('http://localhost:5235/User');
        this.users = response.data;
      } catch (error) {
        console.error('Error fetching users:', error);
      }
    },
    async fetchCampTypes() {
      try {
        const response = await axios.get('http://localhost:5235/Camptype');
        this.campTypes = response.data;
        //console.log('Fetched camp types:', this.campTypes); // Log fetched camp types
      } catch (error) {
        console.error('Error fetching camp types:', error);
      }
    },
    getCampTypeNames(campTypeIds) {
      // Ensure campTypeIds is a plain JavaScript array
      const ids = JSON.parse(JSON.stringify(campTypeIds));

      //console.log('Original campTypeIds:', campTypeIds);
      //console.log('Plain JavaScript array ids:', ids);

      if (!ids || ids.length === 0) {
        return '';
      }

      const names = ids.map(id => {
        const campType = this.campTypes.find(type => type.id === id);
        return campType ? campType.typeName : 'Unknown'; // Ensure to use the correct property for typeName
      });

      //console.log('Mapped camp type names:', names.join(', '));

      return names.join(', ');
    },
    getAmenityName(amenityId) {
      const amenity = this.amenitiesList.find(amenity => amenity.id === amenityId);
      return amenity ? amenity.name : 'Unknown';
    },
    getCampingGroundName(campingGroundId) {
      const ground = this.campingGrounds.find(g => g.id === campingGroundId);
      return ground ? ground.name : 'Unknown';
    },
    getUserName(userId) {
      const user = this.users.find(user => user.id === userId);
      return user ? user.username : 'Unknown';
    }
  }
};
</script>

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

/* Styling for spot details */
.spot-details {
  flex: 1;
}

/* Styling for booking section */
.booking-section {
  flex: 1;
  padding: 10px;
  border-left: 1px solid #ccc;
  border-right: 1px solid #ccc;
  display: flex;
  flex-direction: column;
}

.date-picker-section {
  display: flex;
  align-items: center; /* Center content horizontally */
  margin-bottom: 10px; /* Add margin bottom for spacing */
}

.date-picker-section label {
  margin-right: 10px; /* Add margin between label and input */
}

.time-inputs {
  display: flex;
  flex-direction: column; /* Display time inputs vertically */
  margin-bottom: 10px; /* Add margin bottom for spacing */
}

.time-picker-group {
  display: flex;
  align-items: center; /* Center content vertically */
  margin-bottom: 5px; /* Add margin bottom for spacing between groups */
}

.time-picker-group label {
  margin-right: 10px; /* Add margin between label and input */
}

.book-button-container {
  margin-top: auto; /* Push the button to the bottom */
}

button {
  margin-top: 10px; /* Add space between time pickers and button */
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

button:hover {
  background-color: #45a049;
}

/* Styling for amenities section */
.amenities-section {
  flex: 1;
  padding-left: 10px;
}
</style>
