<template>
  <div>
    <!-- Main container -->
    <div class="main-container">
      
      <!-- Banner container -->
      <div class="banner">
        <h3>All Listings</h3>
      </div>

      <!-- Create Listing container -->
      <div class="create-listing-container">
        <!-- Top section -->
        <div class="create-listing-container-top">
          <!-- Left side -->
          <div class="create-listing-container-left">

            <!-- Name field -->
            <div class="field">
              <label for="name">Name:</label>
              <input type="text" id="name" v-model="newListing.name" class="boxes" required />
            </div>

            <!-- Size picker -->
            <div class="field">
              <label for="size">Size:</label>
              <input type="number" id="size" v-model.number="newListing.size" class="boxes" required />
            </div>

            <!-- Description field -->
            <div class="field">
              <label for="description">Description:</label>
              <input type="text" id="description" v-model="newListing.description" class="boxes" required />
            </div>

            <!-- Price picker -->
            <div class="field">
              <label for="price">Price:</label>
              <input type="number" id="price" v-model.number="newListing.price" class="boxes" required />
            </div>
            
          </div>

          <!-- Right side -->
          <div class="create-listing-container-right">
            <!-- Dropdown boxes -->
            <div class="field">
              <label for="dropdown2">Camping Ground:</label>
              <select id="dropdown2" v-model="newListing.dropdown2" class="boxes">
                <option disabled value="">Please select one</option>
                <option v-for="ground in campingGrounds" :key="ground" :value="ground">{{ ground }}</option>
              </select>
            </div>

            <div class="field">
              <label for="dropdown3">Amenity:</label>
              <multiselect v-model="newListing.dropdown3"
                           :options="amenities"
                           placeholder="Select amenities"
                           :multiple="true"
                           :close-on-select="true"
                           id="dropdown3"
                           class="boxes">
              </multiselect>
            </div>

            <div class="field">
              <label for="dropdown4">Camp Type:</label>
              <multiselect v-model="newListing.dropdown4"
                           :options="campTypes"
                           placeholder="Select camp types"
                           :multiple="true"
                           :close-on-select="true"
                           id="dropdown4"
                           class="boxes">
              </multiselect>
            </div>

            <!-- Availability toggle -->
            <div class="field checkbox-group">
              <label for="isAvailable">Is Available:</label>
              <input type="checkbox" id="isAvailable" v-model="newListing.isAvailable" style="align-self: flex-start;" />
            </div>
          </div>
        </div>

        <!-- Bottom section -->
        <div class="create-listing-container-bottom">
          <!-- Button -->
          <button @click="createListing">Create Listing</button>
        </div>
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
import axios from 'axios';
import Multiselect from 'vue-multiselect';

export default {
  name: 'ListingPage',
  components: {
    Multiselect
  },
  props: {
    userData: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      newListing: {
        name: '',
        size: null,
        description: '',
        price: null,
        isAvailable: true,
        dropdown2: '', // Will be filled with camping grounds names
        dropdown3: [], // Use array for multi-select
        dropdown4: []  // Use array for multi-select
      },
      campingSpots: [],
      campingGrounds: [],
      amenities: [],
      campTypes: [],
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
          this.fetchCampingSpots(),
          this.fetchCampingGrounds(),
          this.fetchAmenities(),
          this.fetchCampTypes()
        ]);
      } catch (error) {
        console.error('Error initializing data:', error);
      } finally {
        this.isLoading = false;
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
    async fetchCampingGrounds() {
      try {
        const response = await axios.get('http://localhost:5235/CampingGround');
        // Extracting names from camping grounds objects
        this.campingGrounds = response.data.map(campingGround => campingGround.name);
      } catch (error) {
        console.error('Error fetching camping grounds:', error);
      }
    },
    async fetchAmenities() {
      try {
        const response = await axios.get('http://localhost:5235/Amenity');
        // Extracting names from amenities objects
        this.amenities = response.data.map(amenity => amenity.name);
      } catch (error) {
        console.error('Error fetching amenities:', error);
      }
    },
    async fetchCampTypes() {
      try {
        const response = await axios.get('http://localhost:5235/Camptype');
        // Extracting type names from camp types objects
        this.campTypes = response.data.map(campType => campType.typeName);
      } catch (error) {
        console.error('Error fetching camp types:', error);
      }
    },
    createListing() {
      // Handle the submission logic here
      console.log('New Listing:', this.newListing);
      // Reset the fields
      this.newListing = {
        name: '',
        size: null,
        description: '',
        price: null,
        isAvailable: true, // Reset isAvailable to true after submission
        dropdown2: '',
        dropdown3: [],
        dropdown4: []
      };
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

/* Styling for create listing container */
.create-listing-container {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
  display: flex;
  flex-direction: column;
}

/* Styling for create listing container top section */
.create-listing-container-top {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

/* Styling for create listing container bottom section */
.create-listing-container-bottom {
  margin-top: 10px;
  text-align: center;
}

/* Styling for create listing container left section */
.create-listing-container-left {
  width: 48%;
}

/* Styling for create listing container right section */
.create-listing-container-right {
  width: 48%;
  display: flex;
  flex-direction: column;
}

/* Styling for field */
.field {
  margin-bottom: 10px;
}

.field label {
  display: block;
  margin-bottom: 5px;
}

.field .boxes {
  width: 40%; 
  padding: 5px;
  box-sizing: border-box;
}

/* Styling for isAvailable checkbox */
.checkbox-group {
  display: flex;
  align-items: center;
}

.checkbox-group label {
  margin-right: 10px; 
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
