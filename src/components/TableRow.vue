<template>
    <div class="row table-row">
        <div class="col">
            <div>{{ name }}</div>
            <div>{{ mac }}</div>
            <div>{{ phone }}</div>
        </div>
    </div>
</template>

<script>
//import "../assets/js/bootstrap.min.js";

export default {
    name: 'TableRow',
    props: {
        name: String,
        mac: String,
        phone: String,
    }
}
</script>