<template>
    <div class="vendor-filter">
        <div class="vendor-filter__header">
            <div class="vendor-filter__title">
                Фильтр:
                <span
                    class="vendor-filter__name"
                    :class="{'vendor-filter__active' : isAlbum}"
                    @click="selectedFilter('isAlbum', 'isFavourite')"
                >
                    По альбомам
                </span>
                <span
                    class="vendor-filter__name"
                    :class="{'vendor-filter__active' : isFavourite}"
                    @click="selectedFilter('isFavourite', 'isAlbum')"
                >
                    Избранное
                </span>
            </div>
        </div>
        <div class="vendor-filter__scroll-wrap">
            <div class="vendor-filter__scroll">
                <div class="vendor-filter__list"
                     v-for="group in getFilteredPhotos"
                     :key="group.group"
                >
                    <h2>{{ group.groupName }}</h2>
                    <div>
                        <popup-image
                            v-for="image in group.data"
                            :key="image.id"
                            :image="image"
                            class="vendor-filter__item"
                            @changeFavourite="saveToLocalStorage()"
                        />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import PopupImage from '@/components/popup-image'

export default {
    data() {
        return {
            arrayPhotos: [],
            isAlbum: false,
            isFavourite: false,
        };
    },
    computed: {
        getFilteredPhotos() {
            if (this.isAlbum) {
                return this.sortedByGroup(this.arrayPhotos);
            } else if (this.isFavourite) {
                return this.sortedByGroup(this.arrayPhotos.filter(photos => photos.favourite), true);
            } else {
                return this.sortedByGroup(this.arrayPhotos, true);
            }
        }
    },
    methods: {
        sortedByGroup(arrayPhoto, sortByFirstLetter) {
            let resultArrayPhoto = []
            arrayPhoto.forEach(photo => {
                let sortParameter = sortByFirstLetter ? photo.title[0].toUpperCase() : photo.albumId;
                let findGroup = resultArrayPhoto.find(group => group.group === sortParameter)
                if (findGroup) {
                    findGroup.data.push(photo)
                } else {
                    resultArrayPhoto.push({
                        group: sortParameter,
                        data: [photo]
                    })
                }
            })
            resultArrayPhoto = resultArrayPhoto.map(photo => {
                return {
                    group: photo.group,
                    data: this.sortByLetter(photo.data, 'title'),
                    groupName: sortByFirstLetter ? photo.group : photo.data[0].title
                }
            })
            this.saveToLocalStorage()
            return this.sortByLetter(resultArrayPhoto, 'groupName')
        },
        selectedFilter(presentFilter, pastFilter) {
            this[presentFilter] = !this[presentFilter];
            this[pastFilter] = false;
        },
        sortByLetter(array, nameSortParameter) {
            return array.sort((a, b) => {
                let firstParameter = a[nameSortParameter];
                let secondParameter = b[nameSortParameter];
                return firstParameter > secondParameter ? 1 : firstParameter < secondParameter ? -1 : 0
            })
        },
        saveToLocalStorage() {
            localStorage.setItem('arrayPhotos', JSON.stringify(this.arrayPhotos))
        },
    },
    components: {
        PopupImage,
    },
    async created() {
        let arrayPhotos = JSON.parse(localStorage.getItem('arrayPhotos'))
        if (arrayPhotos && arrayPhotos.length) {
            this.arrayPhotos = arrayPhotos;
        } else {
            let allArray = await fetch('http://jsonplaceholder.typicode.com/photos').then(response => response.json())
            let uniqueAlbumId = new Set();
            allArray.forEach(photo => {
                if (photo.albumId <= 32 && !uniqueAlbumId.has(photo.albumId)) {
                    uniqueAlbumId.add(photo.albumId)
                    const maxLength = Math.floor(5 + Math.random() * (10 + 1 - 5))
                    this.arrayPhotos = this.arrayPhotos.concat(allArray.filter(item => item.albumId === photo.albumId).slice(0, maxLength));
                }
            })
            this.arrayPhotos = this.arrayPhotos.map(photo => {
                return {
                    ...photo,
                    favourite: false,
                }
            })
            this.saveToLocalStorage();
        }
    },
};
</script>

<style lang="scss">

.vendor-filter {
    font-weight: 700;
    font-family: Open Sans, Roboto, sans-serif;
    width: 1300px;
    height: 600px;
    margin: auto;
    padding: 3.6rem 4.8rem 6rem;
    font-size: 1.2rem;
    background: #ffffff;
    border-radius: 1.2rem;
    box-sizing: border-box;
    overflow: hidden;

    -ms-overflow-style: none;
    scrollbar-width: none;

    ::-webkit-scrollbar {
        width: 0;
        background: transparent;
    }


    &__header {
        margin: 0 0 1.5rem;
    }

    &__title {
        font-weight: bold;
        font-size: 1.4rem;
        line-height: 1.9rem;
    }

    &__name {
        margin: 0 0.5rem;
        text-decoration: underline;
        cursor: pointer;
        padding: .5rem;
    }

    &__active {
        color: white;
        background-color: #545454;
        border-radius: .5rem;
    }

    &__scroll-wrap {
        overflow: auto;
        height: 100%;
    }


    &__scroll {
        column-count: 4;
        column-gap: 50px;
    }

    &__list {
        overflow: hidden;
        -webkit-column-break-inside: avoid;
        page-break-inside: avoid;
        break-inside: avoid;
        break-inside: avoid-column;
    }

    &__item {
        margin-top: .8rem;
    }
}
</style>
