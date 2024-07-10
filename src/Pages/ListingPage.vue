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
              <label for="dropdown2">Dropdown 2:</label>
              <select id="dropdown2" v-model="newListing.dropdown2" class="boxes">
                <option disabled value="">Please select one</option>
                <option value="option1">Option 1</option>
                <option value="option2">Option 2</option>
              </select>
            </div>
            <div class="field">
              <label for="dropdown3">Dropdown 3:</label>
              <select id="dropdown3" v-model="newListing.dropdown3" class="boxes">
                <option disabled value="">Please select one</option>
                <option value="option1">Option 1</option>
                <option value="option2">Option 2</option>
              </select>
            </div>
            <div class="field">
              <label for="dropdown4">Dropdown 4:</label>
              <select id="dropdown4" v-model="newListing.dropdown4" class="boxes">
                <option disabled value="">Please select one</option>
                <option value="option1">Option 1</option>
                <option value="option2">Option 2</option>
              </select>
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

export default {
  name: 'ListingPage',
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
        dropdown1: '',
        dropdown2: '',
        dropdown3: '',
        dropdown4: ''
      },
      campingSpots: [],
      campingGrounds: [],
      users: [],
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
          this.fetchUsers(),
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
        this.campingGrounds = response.data;
      } catch (error) {
        console.error('Error fetching camping grounds:', error);
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
    async fetchAmenities() {
      try {
        const response = await axios.get('http://localhost:5235/Amenity');
        this.amenities = response.data;
      } catch (error) {
        console.error('Error fetching amenities:', error);
      }
    },
    async fetchCampTypes() {
      try {
        const response = await axios.get('http://localhost:5235/Camptype');
        this.campTypes = response.data;
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
        isAvailable: true,
        dropdown1: '',
        dropdown2: '',
        dropdown3: '',
        dropdown4: ''
      };
    }
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
