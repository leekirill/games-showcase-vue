<template>
        <div class="top__line">
            <div class="top__left" @click="toggleFilterList">Filter
                <Transition>
                    <ul v-show="showFilterList" class="filterList">
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
            <div class="top__right" @click="toggleSortList">{{ sortName }}
                <Transition>
                    <ul v-show="sortList" class="sortList" @click="sortData">
                        <li v-for="s, i in sortArr" :key="i" :class="[(activeId === i) ? 'item--active' : 'item']"
                            @click="setActiveId(i)" :aria-label="s.ariaLabel">{{ s.name }}</li>
                    </ul>
                </Transition>
            </div>
        </div>
</template>

<script>
const API_KEY = "bc22c20a2222477ea32564d9ac421797";

export default {
    props: {
        dataProp: {
            type: Array,
        },
    },
    data() {
        return {
            sortArr: [
                { ariaLabel: 'popularity', name: 'By popularity' },
                { ariaLabel: 'asc', name: 'Rating asc' },
                { ariaLabel: 'desc', name: 'Rating desc' }
            ],
            platformsData: [],
            sortName: 'Sort',
            sortList: false,
            showFilterList: false,
            activeId: 0,
        }
    },
    methods: {
        async getPlatformList() {
            const res = await fetch(
                `https://api.rawg.io/api/platforms/lists/parents?key=${API_KEY}`
            );
            const platformsData = await res.json()
            this.platformsData = platformsData.results.sort((a, b) => a.id - b.id)
            console.log(this.platformsData)
        },
        toggleFilterList() {
            this.showFilterList = !this.showFilterList
        },
        toggleSortList() {
            this.sortList = !this.sortList
        },
        getCheckedFilterItems(e) {
            this.$emit('getCheckedFilterItems', e)
        },
        getFilteredData() {
            this.$emit('getFilteredData')
        },
        getSearchQuery(e) {
            this.$emit('searchQuery', e.target.value)
        },
        sortData(e) {
            this.$emit('sortData', e)
            this.sortName = `Sort by: ${e.target.textContent}`
        },
        setActiveId(i) {
            this.activeId = i
            return this.activeId === i
        }
    },
    mounted() {
        this.getPlatformList()
    }
}
</script>

<style scoped lang="scss">
.top {
    &__line {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0 20px;
        margin-bottom: 20px;
        border: 1px solid #a9a9a9;
    }
    &__left, &__right {
        min-width: 100px;
        padding: 16px 0;
        border: 1px solid red;
        &:hover {
            cursor: pointer;
        }
    }
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

    .search {
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