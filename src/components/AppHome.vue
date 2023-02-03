<template>
    <h1>Page {{ currentPage }}</h1>
        <div class="top__line">
            <div @mouseenter="toggleFilterList" @mouseleave="toggleFilterList">Filter
                <Transition>
                    <ul v-show="filterList" class="filterList">
                        <li class="item">By popularity</li>
                        <li class="item">Rating asc</li>
                        <li class="item">Rating desc</li>
                    </ul>
                </Transition>
            </div>
            <div @mouseenter="toggleSortList" @mouseleave="toggleSortList">{{sortName}}
                <Transition>
                <ul v-show="sortList" class="sortList" @click="sortData"> 
                    <li v-for="s, i in sortArr" :key="i" :class="[(activeId === i) ? 'item--active' : 'item']" @click="setActiveId(i)" :aria-label="s.ariaLabel">{{ s.name }}</li>
                </ul>
                </Transition>
            </div>
        </div>
        <h2 v-if="isLoading">Loading</h2>
        <ul v-else class="list">
            <li class="list__item" v-for="d,i in data" :key={i}>
                <img :src="d.background_image" :alt="d.name">
                <strong>{{ d.name }}</strong>
                <p>{{ d.rating }}</p>
            </li>
        </ul>
        <div class="pagination">
            <button @click="prevPage">←</button>
            <button @click="changePage" v-for="item,i in maxPages" :key='i'>{{ item }}</button>
            <button @click="nextPage">→</button>
        </div>
</template>

<script>

const API_KEY = "bc22c20a2222477ea32564d9ac421797";

export default {
    data() {
        return {
            data: [],
            sortArr: [
                { ariaLabel: 'popularity', name: 'By popularity' },
                { ariaLabel: 'asc', name: 'Rating asc' },
                { ariaLabel: 'desc', name: 'Rating desc' }
            ],
            currentPage: 1,
            activeId: 0,
            maxPages: null,
            sortList: false,
            sortName: 'Sort',
            filterList: false,
            isLoading: false,
            baseUrl: 'http://localhost:8080/'
        }
    },
    methods: {
        async getData() {
            this.isLoading = true

            const res = await fetch(
                `https://api.rawg.io/api/games?key=${API_KEY}&dates=2019-09-01,2019-09-30&platforms=18,1,7&page=${this.currentPage}`
            );
            const dataRes = await res.json()
            this.data = dataRes.results
            this.maxPages = Math.ceil(dataRes.count / dataRes.results.length)
            this.isLoading = false
        },
        nextPage() {
            this.currentPage++
            this.getData()
            this.changeUrl()
        },
        prevPage() {
            if (this.currentPage === 1) {
                return
            }
            this.currentPage--
            this.getData()
            this.changeUrl()
        },
        changePage(e) {
            this.currentPage = parseInt(e.target.innerHTML)
            this.getData()
            this.changeUrl()
        },
        setActiveId(i) {
            this.activeId = i
            return this.activeId === i
        },
        changeUrl() {
            let url = new URL(`/?query=${this.currentPage}`, this.baseUrl)
            history.pushState(null, '', url)

            if (this.currentPage === 1) {
                this.resetUrl()
            }
        },
        resetUrl() {
            let url = new URL('/', this.baseUrl)
            history.pushState(null, '', url)
        },
        toggleFilterList() {
            this.filterList = !this.filterList
        },
        toggleSortList() {
            this.sortList = !this.sortList
        },
        sortData(e) {
            if (e.target.nodeName === "LI") {
                console.log(e.target)

                if (e.target.ariaLabel === 'desc') {
                    this.data = this.data.sort((a, b) => b.rating - a.rating)
                    this.sortName = e.target.innerHTML
                }
                if (e.target.ariaLabel === 'asc') {
                    this.data = this.data.sort((a, b) => a.rating - b.rating)
                    this.sortName = e.target.innerHTML
                }
                if (e.target.ariaLabel === 'popularity') {
                    this.data = this.data.sort((a, b) => a.reviews_count - b.reviews_count)
                    this.sortName = e.target.innerHTML
                }
            }

        }

    },
    mounted() {
        this.getData()
        this.resetUrl()
    },
}
</script>

<style scoped lang="scss">
.list {
        display: grid;
        grid-template-columns: 1fr 1fr 1fr 1fr;
        gap: 20px;
        padding: 0;
        margin: 0
    }
    .list__item {
        display: flex;
        flex-direction: column;
        list-style: none;
        strong {
            margin-top: 14px;
        }
        p {
            padding: 0;
            margin: 0;
        }
    }
    img {
        width: 100%;
        height: 160px;
        object-fit: cover;
    }
    .top__line {
        display: flex;
        justify-content: space-between;
        padding: 16px 20px;
        margin-bottom: 20px;
        border: 1px solid #a9a9a9;
    }
    .filterList {
        padding: 0;
        position: absolute;
        background-color: #171717;
        color: #fff;
        border: 1px solid #fff;

        .item {
            list-style: none;
            padding: 20px
        }
    }
    .sortList {
        padding: 0;
        position: absolute;
        background-color: #171717;
        color: #fff;
        border: 1px solid #fff;

        .item {
            list-style: none;
            padding: 20px;
            cursor: pointer;
            &--active {
                @extend .item;
                color: red;
            }
        }
    }
    .pagination {
        display: flex;
        gap: 5px;
        justify-content: center;
        margin-top: 60px;
            button {
                background: #171717;
                border: 1px solid #a9a9a9;
                min-width: 40px;
                height: 40px;
                transition: 250ms;
                color: #fff;
                &:hover {
                    cursor: pointer;
                    border: 1px solid rgb(106, 106, 106)
                }
            }
    }

    .v-enter-active,
    .v-leave-active {
        transition: opacity 0.5s ease;
    }

    .v-enter-from,
    .v-leave-to {
        opacity: 0;
    }
</style>