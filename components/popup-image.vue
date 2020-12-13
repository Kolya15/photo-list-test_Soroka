<template>
    <div class="image">
        <img :src="image.url" class="image-src">
        <h3 class="image-title">{{image.title}}</h3>
        <img
            v-if="!image.favourite"
            src="@/static/img/favourite.svg"
            class="image-icon__favourite"
            alt="favourite"
            @click="changeFavourite()"
        >
        <img
            v-else
            src="@/static/img/cross.svg"
            class="image-icon__favourite"
            alt="remove"
            @click="changeFavourite()"
        >
    </div>
</template>

<script>

export default {
    name: "popup-image",
    props: {
        image: Object,
    },
    methods:{
        changeFavourite(){
            this.image.favourite = !this.image.favourite;
            this.$emit('changeFavourite');
        }
    }
}
</script>

<style scoped lang="scss">

.image {
    display: flex;
    align-items: center;
    width: 260px;
    height: 40px;
    background: #F7F8F9;
    border-radius: 8px;
    padding: 4px;
    position: relative;

    &-src {
        width: 32px;
        height: 32px;
        border-radius: 8px;
        margin-right: 11px;
     }

    &-title {
        width: 70%;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        margin-right: 10px;
    }

    &-icon__favourite {
        float: right;
        width: 15px;
        height: 15px;
        cursor: pointer;
    }
}

</style>