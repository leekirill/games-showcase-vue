<template>
    <h1><a href="/">Page {{ currentPage }}</a></h1>
        <div class="top__line">
            <div @mouseenter="data && toggleFilterList" @mouseleave="toggleFilterList">Filter
                <Transition>
                    <ul v-show="filterList" class="filterList">
                        <li v-for="p, i in platformsData" :key="i" class="item">
                            <input :id="p.id" type="checkbox" @input="getCheckedFilterItems" class="item__checkbox">
                            <label :for="p.id">{{ p.name }}</label>
                        </li>
                        <button @click="getFilteredData" class="filterBtn">Save</button>
                    </ul>
                </Transition>
            </div>
            <div>
                <input type="text" name="" placeholder="Search" class='search' @change="getSearchQuery">
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
            <button @click="changePage" :class="[(activeId === i) ? 'pag--active' : 'pag']" v-for="item,i in 10" :key='i'>{{ item }}</button>
            <button class="pag">...</button>
            <button @click="changePage" :class="[(activeId === i) ? 'pag--active' : 'pag']">{{ this.maxPages }}</button>

            <button @click="nextPage">→</button>
        </div>
</template>

<script>

const API_KEY = "bc22c20a2222477ea32564d9ac421797";

export default {
    data() {
        return {
            data: [],
            platformsData: [],
            filterItems: [],
            searchQuery: '',
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
                `https://api.rawg.io/api/games?key=${API_KEY}${this.getFilteredPlatforms}&page=${this.currentPage}&search=${this.searchQuery}`
            );
            const dataRes = await res.json()
            this.data = dataRes.results
            this.pagination(dataRes)
        },
        async getPlatformList() {
            const res = await fetch(
                `https://api.rawg.io/api/platforms/lists/parents?key=${API_KEY}`
            );
            const platformsData = await res.json()
            this.platformsData = platformsData.results.sort((a,b) => a.id - b.id)
        },  
        pagination(dataRes) {
            this.maxPages = Math.ceil(dataRes.count / dataRes.results.length)
            this.isLoading = false
        },
        actualPagesCount() {
            return 10
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
            this.setActiveId(this.currentPage - 1)
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
        },
        getCheckedFilterItems(e) {
            if (e.target.checked) {
                this.filterItems.push(parseInt(e.target.id))
            } else {
                this.filterItems = this.filterItems.filter(item => item !== parseInt(e.target.id))
            }
            console.log(this.filterItems)
        },
        getFilteredData() {
            console.log(this.filterItems.join(','))
            this.getData()
            this.activeId = 0
            this.currentPage = 1
        },
        getSearchQuery(e) {
            this.searchQuery = e.target.value
            this.getData()
            this.activeId = 0
            this.currentPage = 1
        }
    },
    computed: {
        getFilteredPlatforms() {
            console.log(this.filterItems.join(','))
            return this.filterItems ?? '&platforms=' + this.filterItems.join(',')
        }
    },
    mounted() {
        this.getData()
        this.resetUrl()
        this.getPlatformList()
    }
}
</script>

<style scoped lang="scss">
a {
    text-decoration: none;
    color: #fff;
}
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
        align-items: center;
        padding: 8px 20px;
        margin-bottom: 20px;
        border: 1px solid #a9a9a9;
    }
    .filterList {
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;
        padding: 0;
        position: absolute;
        background-color: #171717f4;
        color: #fff;
        border: 1px solid #fff;

        .item {
            list-style: none;
            padding: 20px;
            text-align: left;
        }
        .filterBtn {
            border-top: 1px solid #fff;
            border-left: 1px solid #fff;
            background-color: transparent;
            color: #fff;
            cursor: pointer;
            transition: 250ms;
            &:hover {
                background-color: black;
            }
        }
    }
    .sortList {
        padding: 0;
        position: absolute;
        background-color: #171717f4;
        color: #fff;
        border: 1px solid #fff;

        .item {
            list-style: none;
            padding: 20px;
            cursor: pointer;
            &--active {
                @extend .item;
                color: #ffffff;
                font-weight: 700;
                background: rgba(255, 255, 255, .1);
            }
        }
    }
    .pagination {
        display: flex;
        gap: 5px;
        justify-content: center;
        margin-top: 60px;
        .pag {
            color: #fff
        }
        .pag--active {
            color: #ffffff;
            font-weight: 700;
            background: rgba(255, 255, 255, .1);
        }
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

    .item__checkbox {
        position: absolute;
        z-index: -1;
        opacity: 0;
    }
    .item__checkbox+label {
        display: inline-flex;
        align-items: center;
        user-select: none;
        cursor: pointer;
    }

    .item__checkbox+label::before {
        content: '';
        display: inline-block;
        width: 1em;
        height: 1em;
        flex-shrink: 0;
        flex-grow: 0;
        border: 1px solid #adb5bd;
        margin-right: 0.5em;
        background-repeat: no-repeat;
        background-position: center center;
        background-size: 50% 50%;
    }
    .item__checkbox:checked+label::before {
        border-color: #fff;
        background-color: transparent;
        background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 8 8'%3e%3cpath fill='%23fff' d='M6.564.75l-3.59 3.612-1.538-1.55L0 4.26 2.974 7.25 8 2.193z'/%3e%3c/svg%3e");
    }

    .search  {
        background: none;
        color: #fff;
        outline: none;
        border: 1px solid #545454;
        padding: 8px 10px;
        min-width: 400px;
        transition: 250ms;
        &:hover {
            border: 1px solid #a9a9a9;
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