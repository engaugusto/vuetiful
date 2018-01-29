<template>
    <div class="paginator">
        <div class="paginator-body">
            <slot :data="data" :page-number="pageNumber"></slot>
        </div>
        <div class="paginator-footer" layout="row center-justify">
            <div class="paginator-page-numbers">
                <div class="paginator-button paginator-page-first" v-if="showFirst()" @click="moveTo(1)">
                    <span>&lt;&lt;</span>
                </div>
                <div class="paginator-button paginator-page-previous" v-if="showPrevious()" @click="movePrevious">
                    <span>&lt;</span>
                </div>
                <div v-bind:key="num" class="paginator-button paginator-page-number" :class="{ 'active': num === pageNumber }" v-for="num in pages.length" @click="moveTo(num)" >
                    <span>{{ num }}</span>
                </div>
                <div class="paginator-button paginator-page-next" v-if="showNext()" @click="moveNext">
                    <span>&gt;</span>
                </div>
                <div class="paginator-button paginator-page-last" v-if="showLast()"  @click="moveTo(pages.length)">
                    <span>&gt;&gt;</span>
                </div>
            </div>
            <div class="size" v-if="pages.length > 0">
                {{(pageNumber-1)*pageSize+1}} até {{ lastRegister }} do total de {{source.length}}
            </div>            
        </div>
    </div>
</template>

<script>
    import filterBy from "../../utilities/filter-by.js";
    import page from "../../utilities/page.js";

    export default {

        props: {

            source: {
                type: Array,
                default: () => []
            },

            pageSize: {
                type: Number,
                default: 25
            },

            filter: {
                type: String
            }

        },

        data() {
            return {
                index: 0
            };
        },

        computed: {

            pages() {
                let data = this.source;

                if (this.filter) {
                    data = filterBy(data, this.filter);
                }

                let pages = page(data, this.pageSize);

                // need to reset the page number if the data length changes
                // otherwise the index will be outside the bounds of the data
                if (this.pageNumber > pages.length) {
                    this.pageNumber = 1;
                }

                return pages;
            },

            pageNumber: {
                get() {
                    return this.index + 1;
                },
                set(value) {
                    this.index = value - 1;
                    this.$emit("page-changed", value);
                }
            },

            //added to get the beggining of the registers
            lastRegister: {
                get(){
                    if(this.index==this.pages.length-1)return ((this.index)*this.pageSize)+(this.data.length%this.pageSize);
                    return ((this.index+1)*this.pageSize);
                }
            },

            data() {
                return this.pages[this.index];
            }

        },

        methods: {
            showFirst(){
                return this.pageNumber > 1;
            },
            showPrevious(){
                return this.pageNumber > 1;
            },

            showNext() {
                return this.pageNumber < this.pages.length;
            },
            showLast() {
                 return this.pageNumber < this.pages.length;
            },

            movePrevious() {
                this.pageNumber -= this.pageNumber > 1 ? 1 : 0;
            },
            moveNext() {
                this.pageNumber += (this.pageNumber != this.pages.length) ? 1 : 0;
            },

            moveTo(pageNumber) {
                if (pageNumber > 0 && pageNumber <= this.pages.length) {
                    this.pageNumber = pageNumber;
                }
            }

        }

    }
</script>

<style lang="scss">
    @import "../../assets/styles/abstract/_variables.scss";

    .paginator {
        border: 1px solid $colour-border;
        border-radius: $border-radius;
    }

    .paginator-footer {
        padding: 1rem;
        background-color: $colour-background-medium;
        border-top: 1px solid $colour-border;
    }

    .paginator-button {
        display: inline-block;
        min-width: 1.5em;
        padding: 0 0.5rem;
        font-weight: 600;
        background-color: $colour-border;
        border-radius: $border-radius;
        cursor: pointer;

        &.active {
            color: $colour-font-negative;
            background-color: $colour-primary;
        }
    }

    .paginator-page-number {
        margin: 0 0.25rem;
    }

</style>