<template>
    <div>
        <ul>
            <li v-for="d,i in counts" :key={i}>{{ d }}</li>
        </ul>
    </div>
</template>

<script>
export default {
    data() {
        return {
            counts: [1,3,4]
        }
    },
}
</script>