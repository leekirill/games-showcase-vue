<template>
    <h1><a href="/">Page {{ currentPage }}</a></h1>
        <app-header :activeId="activeId" @searchQuery="getSearchQuery" @sortData="sortData" @getCheckedFilterItems="getCheckedFilterItems" @getFilteredData="getFilteredData"></app-header>
    <div v-if="isLoading">
      <rotate-loader label="Loading..." color="#fff"></rotate-loader>
    </div>        
    <ul v-else class="list">
            <li class="list__item" v-for="d,i in data" :key="i" @click="openItem(d.id)">
                <img :src="d.background_image" :alt="d.name">
                <strong>{{ d.name }}</strong>
                <p>{{ d.rating }}</p>
            </li>
        </ul>
        <div class="pagination">
            <button @click="prevPage">←</button>
            <button @click="changePage" :class="[(activeId === i) ? 'pag--active' : 'pag']" v-for="item,i in 10" :key='i'>{{ item }}</button>
            <button class="pag">...</button>
            <button @click="changePage" class="pag">{{ this.maxPages }}</button>

            <button @click="nextPage">→</button>
        </div>
</template>

<script>
import AppHeader from './AppHeader.vue'
import RotateLoader from 'vue-spinner/src/RotateLoader.vue'

const API_KEY = "bc22c20a2222477ea32564d9ac421797";

export default {
    data() {
        return {
            data: [],
            filterItems: [],
            itemId: 0,
            currentPage: 1,
            activeId: 0,
            searchValue: '',
            maxPages: null,
            isLoading: false,
            baseUrl: 'http://localhost:8080/'
        }
    },
    components: {
        AppHeader,
        RotateLoader 
    },
    methods: {
        async getData() {
            this.isLoading = true

            const res = await fetch(
                `https://api.rawg.io/api/games?key=${API_KEY}${this.getFilteredPlatforms}${this.getSearchedData}&page=${this.currentPage}`
            );
            const dataRes = await res.json()
            this.data = dataRes.results
            this.pagination(dataRes)
        },
        async getItemData() {
            const res = await fetch(
                `https://api.rawg.io/api/games/${this.itemId}?key=${API_KEY}`
            );
            const dataRes = await res.json()
            console.log(dataRes)
        }, 
        openItem(i) {
            console.log(i)
            this.itemId = i
            this.getItemData()
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
        getSearchQuery(e) {
            this.searchValue = e

            this.getData()
            this.activeId = 0
            this.currentPage = 1
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
            this.getData()
            this.activeId = 0
            this.currentPage = 1
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
            this.activeId = 0
            this.currentPage = 1
        }
    },
    computed: {
        getFilteredPlatforms() {
            return this.filterItems.length > 0 ? '&platforms=' + this.filterItems.join(',') : ''
        },
        getSearchedData() {
            return this.searchValue.length > 0 ? '&search=' + this.searchValue : ''
        }
    },
    mounted() {
        this.getData()
        this.resetUrl()
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
        cursor: pointer;
        transition: 250ms;
        &:hover {
            background-color:#2e2e2e;
        }
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
    
    
    .pagination {
        display: flex;
        gap: 5px;
        justify-content: center;
        margin-top: 60px;
        .pag {
            color: #fff
        }
        .pag--active {
            color: #fff;
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
</style>