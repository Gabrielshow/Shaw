<template>
    <div class="centered">
        <div class="inherit">
            <img src="@/assets/Rogers.jpg" alt="logo" class="logo" />
        </div>

        <button class="rogers-blue" @click="redirect">CLICK HERE TO VIEW YOUR CURRENT BILL</button>

        <div class="footer"> 
            <p>&copy; 2025 Rogers Communications</p>
            <p>All Rights Reserved</p>
        </div>
    </div>
</template>

<script setup>
import { useRouter } from 'vue-router';

const router = useRouter(); // Call useRouter here

function redirect() {
    router.push('/form'); // Now router is defined
}
</script>

<style scoped>
.centered {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100vh; 
    box-sizing: border-box;
}

.inherit {
    width: 100%;
    max-width: 600px; /* Set a max width for the image container */
    padding: 20px;
}

.logo {
    width: 100%; 
    height: auto; /* Maintain aspect ratio */
    max-height: 60vh; /* Limit the height to 60% of the viewport height */
    object-fit: contain; /* Ensure the entire image is visible */
}

.rogers-blue {
    width: 100%; 
    max-width: 400px; 
    color: white;
    background: linear-gradient(to bottom, #0082bb 0%, #0074a7 100%); /* Corrected gradient syntax */
    padding: 10px; /* Increased padding for better touch area */
    border-radius: 20px;
    outline: none;
    border: none;
    cursor: pointer;
    text-align: center; /* Center text */
    transition: background-color 0.3s; /* Smooth transition for hover effect */
}

.rogers-blue:hover {
    background: linear-gradient(to bottom, #0074a7 0%, #006699 100%); /* Darker gradient on hover */
}

.footer {
    margin-top: 20px; 
    text-align: center; 
    color: black;
}
</style>