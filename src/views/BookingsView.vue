<script setup>

// import { ref, onMounted } from "vue";
import { ref, onMounted, watch } from "vue" // add watch

const search = ref("");

watch(() => search.value, () => {
    loadAsyncData();
});
const data = ref([]);
const total = ref(0);
const loading = ref(false);
const sortField = ref("vote_count");
const sortOrder = ref("desc");
const defaultSortOrder = ref("desc");
const page = ref(1);
const perPage = ref(20);

const loadAsyncData = () => {
    const params = [
        // "api_key=bb6f51bef07465653c3e553d6ab161a8",
        // "language=en-US",
        // "include_adult=false",
        // "include_video=false",
        `sort_by=${sortField.value}.${sortOrder.value}`, // sort data
        `page=${page.value}`,
        //`email=${search.value},`
    ].join("&");
    loading.value = true;
    fetch(`api/bookings?${params}`) // change url
        .then((response) => response.json())
        .then((result) => {
            perPage.value = result.perPage;
            total.value = result.total;
            data.value = result.bookings

            loading.value = false;
        })
};

/*
 * Handle page-change event
 */
const onPageChange = (p) => {
    page.value = p;
    loadAsyncData();
};

/*
 * Handle sort event
 */
const onSort = (field, order) => {
    sortField.value = field;
    sortOrder.value = order;
    loadAsyncData();
};

/*
 * Type style in relation to the value
 */
const type = (value) => {
    const number = parseFloat(value);
    if (number < 6) {
        return "is-danger";
    } else if (number >= 6 && number < 8) {
        return "is-warning";
    } else if (number >= 8) {
        return "is-success";
    }
};

onMounted(() => {
    loadAsyncData();
});
</script>

<template>
    <section>
        <input class="form-control" v-model="search" placeholder="Search..." />
        <o-table :data="data" :loading="loading" paginated backend-pagination :total="total" :per-page="perPage"
            backend-sorting :default-sort-direction="defaultSortOrder" :default-sort="[sortField, sortOrder]"
            aria-next-label="Next page" aria-previous-label="Previous page" aria-page-label="Page"
            aria-current-label="Current page" @page-change="onPageChange" @sort="onSort">
            <o-table-column v-slot="props" field="email" label="email" numeric sortable>
                <!-- {{ props.row.email }}  -->
                <RouterLink :to="`/booking/${props.row._id}`"> {{ props.row.email }} </RouterLink>
            </o-table-column>
            <o-table-column v-slot="props" field="numTickets" label="#" numeric sortable>
                <span class="tag" :class="type(props.row.numTickets)">
                    {{ props.row.numTickets }}
                </span>
            </o-table-column>
            <o-table-column v-slot="props" field="superhero" label="superhero" numeric sortable>
                {{ props.row.superhero }}
            </o-table-column>
            <o-table-column v-slot="props" field="release_date" label="Modified At" sortable centered>
                {{
            props.row.modified_at
                ? new Date(props.row.modified_at).toLocaleDateString()
                : "unknown"
        }}
            </o-table-column>
            <o-table-column v-slot="props" label="Overview" width="500">
                {{ props.row.overview }}
            </o-table-column>
        </o-table>
    </section>
</template>
