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
                  <label for="start-time">Start Time:</label>
                  <input type="time" id="start-time" v-model="spot.startTime">
                  <label for="end-time">End Time:</label>
                  <input type="time" id="end-time" v-model="spot.endTime">
                </div>
                <button>Book</button>
              </div>

              <!-- Right side for amenities -->
              <div class="amenities-section">
                <h3>Amenities</h3>
                <ul>
                  <li v-for="amenity in spot.amenities" :key="amenity.id">{{ amenity }}</li> <!-- Display amenity names -->
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
import Multiselect from 'vue-multiselect'; // Import vue-multiselect component

export default {
  name: 'CampingSpotPage',
  components: {
    Multiselect // Register Multiselect component
  },
  data() {
    return {
      amenitiesList: [], // Array to store amenities fetched from API
      selectedAmenities: [], // Array to store selected amenities for filtering
      campingGrounds: [], // Array to store camping grounds fetched from API
      campingSpots: [], // Array to store camping spots fetched from API
      users: [], // Array to store users fetched from API
      isLoading: true // Loading state
    };
  },
  mounted() {
    this.initializeData(); // Initialize data fetching
  },
  methods: {
    async initializeData() {
      try {
        await this.fetchAmenities(); // Fetch amenities first
        await Promise.all([this.fetchCampingGrounds(), this.fetchUsers()]); // Fetch camping grounds and users in parallel
        await this.fetchCampingSpots(); // Fetch camping spots after amenities are fetched
      } catch (error) {
        console.error('Error initializing data:', error);
      } finally {
        this.isLoading = false; // Set loading to false once data is fetched
      }
    },
    async fetchAmenities() {
      try {
        // Fetch amenities from your API
        const response = await axios.get('http://localhost:5235/Amenity');
        this.amenitiesList = response.data; // Populate the amenities list
      } catch (error) {
        console.error('Error fetching amenities:', error);
      }
    },
    async fetchCampingGrounds() {
      try {
        // Fetch camping grounds from your API
        const response = await axios.get('http://localhost:5235/CampingGround');
        this.campingGrounds = response.data; // Populate the camping grounds list
      } catch (error) {
        console.error('Error fetching camping grounds:', error);
      }
    },
    async fetchCampingSpots() {
      try {
        // Fetch camping spots from your API
        const response = await axios.get('http://localhost:5235/CampingSpot');
        this.campingSpots = response.data; // Populate the camping spots list

        // Map amenityIds to their corresponding names
        this.campingSpots.forEach(spot => {
          if (spot.amenityIds && Array.isArray(spot.amenityIds)) {
            spot.amenities = spot.amenityIds.map(amenityId => {
              const amenity = this.amenitiesList.find(a => a.id === amenityId);
              return amenity ? amenity.name : 'Unknown';
            });
          } else {
            spot.amenities = [];
          }

          // Find and set the user's name for each camping spot
          const user = this.users.find(u => u.id === spot.userId);
          spot.userName = user ? user.name : 'Unknown';
        });
      } catch (error) {
        console.error('Error fetching camping spots:', error);
      }
    },
    async fetchUsers() {
      try {
        // Fetch users from your API
        const response = await axios.get('http://localhost:5235/User');
        this.users = response.data; // Populate the users list
      } catch (error) {
        console.error('Error fetching users:', error);
      }
    },
    getCampingGroundName(campingGroundId) {
      const ground = this.campingGrounds.find(g => g.id === campingGroundId);
      return ground ? ground.name : 'Unknown';
    },
    getUserName(userId) {
      //console.log('Getting user name for userId:', userId);
      const user = this.users.find(user => user.id === userId);
      //console.log('Found user:', user);
      return user ? user.username : 'Unknown';
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
  align-items: flex-start; /* Center content horizontally */
}

.date-picker-section {
  display: flex;
  align-items: center; /* Center content horizontally */
}

.date-picker-item,
.time-inputs {
  margin-bottom: 10px;
}

.time-inputs {
  display: flex;
  flex-direction: column;
  align-items: center; /* Center content horizontally */
}

.time-input label {
  margin-right: 10px;
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
