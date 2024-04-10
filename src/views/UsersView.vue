<script setup>
import { onMounted, ref } from "vue";
import { jwtDecode } from "jwt-decode";
const users = ref([]);
const name = ref("");

const loadAsyncData = async () => {
    try {
        // Get the token from local storage    
        const token = localStorage.getItem('token');
        // decode jwt token
        const decoded = jwtDecode(token);
        console.log(decoded);
        name.value = `${decoded.first_name} ${decoded.last_name}`;
        // Send a request to the endpoint with the token in the Authorization header
        var response = await fetch("/api/users/with/bookings", {
            headers: {
                Authorization: `Bearer ${token}`,
            }
        });

        if (!response.ok) {
            throw new Error(response.statusText);
        }

        const logout = function () {
            localStorage.removeItem('token');
            location.reload()
        }

        // convert the response to json
        const json = await response.json();
        // log the json
        console.log(json);
        // set the data
        users.value = json;
    } catch (error) {
        console.log(error);
    }
};

onMounted(() => {
    loadAsyncData();
});
</script>

<template>
    {{ name }}
    <div v-for="user in users" :key="user._id">
        {{ user }}
    </div>
    <div v-if="route.name == 'view-booking'">
    <h1>{{ booking.email }}</h1>
    <p>Number of Tickets: {{ booking.numTickets }}</p>
    <p>Payment Method: {{ booking.payment }}</p>
    <p>Favourite Team: {{ booking.team }}</p>
    <p>Favourite Hero: {{ booking.superhero }}</p>
    <p>Terms and Conditions: {{ booking.terms }}</p>
    <p>Last Modified: {{ differenceInDays(new Date(), new Date(booking.modified_at)) }} ago.</p>
</div>
</template>