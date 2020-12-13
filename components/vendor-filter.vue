<template>
    <div class="vendor-filter">
        <div class="vendor-filter__header">
            <div class="vendor-filter__title">
                Фильтр:
                <span
                    class="vendor-filter__name"
                    @click="selectedFilter('isAlbum', 'isFavourite')"
                >
                    По альбомам
                </span>
                <span
                    class="vendor-filter__name"
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
                    <div class="vendor-filter__item">
                        <popup-image
                            v-for="image in group.data"
                            :key="image.id"
                            :image="image"
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
            }
            else if (this.isFavourite) {
                return this.sortedByGroup(this.arrayPhotos.filter(item => item.favourite), true);
            } else {
                return this.sortedByGroup(this.arrayPhotos, true);
            }
        }
    },
    methods: {
        sortedByGroup(arrayPhoto, sortByFirstLetter) {
            let resultArrayPhoto = []
            arrayPhoto.forEach(item => {
                let sortParameter = sortByFirstLetter ? item.title[0].toUpperCase() : item.albumId;
                let findGroup = resultArrayPhoto.find(group => group.group === sortParameter)
                if (findGroup) {
                    findGroup.data.push(item)
                } else {
                    resultArrayPhoto.push({
                        group: sortParameter,
                        data: [item]
                    })
                }
            })
            resultArrayPhoto = resultArrayPhoto.map(item => {
                return {
                    group: item.group,
                    data: this.sortByLetter(item.data, 'title'),
                    groupName: sortByFirstLetter ? item.group : item.data[0].title
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
            this.arrayPhotos = arrayPhotos
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
            this.arrayPhotos = this.arrayPhotos.map(item => {
                return {
                    ...item,
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
    overflow-y: scroll;

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
    }

    &__scroll-wrap {
        overflow: hidden;
    }

    &__scroll {
        //column-count: 4;
        //column-gap: 25px;
        //column-fill: auto;
        //@media (max-width: 1200px) {
        //    column-count: 3;
        //}
        overflow-y: auto;
        column-count: 4;
        column-gap: 25px;
        @media (max-width: 1200px) {
            column-count: 3;
        }
    }

    &__list {
        height: auto;

    }
}
</style>
