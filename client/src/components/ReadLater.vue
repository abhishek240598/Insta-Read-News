<template>
    <v-container grid-list-md text-xs-center>
        <EmptyFeed
            v-if="readingList.length <= 0 && !loading"
            heading="No results found"
            description="I'm sorry but it looks like you have not added anything to read later. Add a few then check back."
        >
        </EmptyFeed>
        <v-layout row wrap>
            <NewsCard
                v-for="(item, index) in readingList"
                :item="item"
                :viewNews="viewNews"
                :key="item.hash"
                :index="index"
                :className="'one-fifth'"
            >
                <div class="one-fifth" slot="slot_0">
                    <v-btn
                        class="blue--text"
                        flat
                        icon
                        @click.stop="viewNews(item, index);saveNewsAsReadhistory(item, index)"
                    >
                        <v-icon class="blue--text">fa-eye</v-icon>
                    </v-btn>
                </div>
                <div class="one-fifth" slot="slot_4">
                    <v-btn
                        class="deep-orange--text"
                        flat
                        icon
			@click.stop="saveNewsAsReadhistory(item, index)"
                        target="_blank"
                        :href="item.URL"
                        rel="noopener"
                    >
                        <v-icon class="deep-orange--text">fa-external-link</v-icon>
                    </v-btn>
                </div>
                <div class="one-fifth" slot="slot_2">
                    <v-btn
                        class="pink--text"
                        flat
                        icon
                        @click.stop="saveNewsAsNotinterested(item, index)"
                    >
                        <v-icon class="pink--text">
                            {{ item.notinterested ? 'fa-thumbs-down' : 'fa-thumbs-o-down' }}
                        </v-icon>
                    </v-btn>
                </div>
                <div class="one-fifth" slot="slot_1">
                    <v-btn
                        class="pink--text"
                        flat
                        icon
                        @click.stop="saveNewsAsFavourite(item, index)"
                    >
                        <v-icon class="pink--text">
                            {{ item.favourite ? 'fa-heart' : 'fa-heart-o' }}
                        </v-icon>
                    </v-btn>
                </div>
                <div class="one-fifth" slot="slot_3">
                    <v-btn
                        class="red--text"
                        flat
                        icon
                        @click.stop="deleteNewsFromReadingList(item)"
                    >
                        <v-icon class="red--text">fa-trash-o</v-icon>
                    </v-btn>
                </div>
            </NewsCard>
        </v-layout>
        <v-progress-circular
            indeterminate
            class="green--text"
            v-if="loading"
            style="margin-top: 21px"
        >
        </v-progress-circular>
        <NewsView
            :showModal="showNewsModal"
            :item="selectedNews"
            :closeModal="closeNewsModal"
        >
            <v-btn
                slot="slot_3"
                flat
                class="btn--active"
                @click.stop="saveNewsAsNotinterested(selectedNews, selectedNewsIndex)"
            >
                <v-icon class="pink--text">
                    {{ selectedNews.notinterested ? 'fa-thumbs-down' : 'fa-thumbs-o-down' }}
                </v-icon>
            </v-btn>
            <v-btn
                slot="slot_2"
                flat
                class="pink--text"
                :input-value="true"
                @click.stop="saveNewsAsFavourite(selectedNews, selectedNewsIndex)"
            >
                <v-icon class="pink--text">
                    {{ selectedNews.favourite ? 'fa-heart' : 'fa-heart-o' }}
                </v-icon>
            </v-btn>
            <v-btn
                slot="slot_3"
                flat
                class="red--text"
                :input-value="true"
                @click.stop="deleteNewsFromReadingList(selectedNews)"
            >
                <v-icon class="red--text">
                    fa-trash-o
                </v-icon>
            </v-btn>
        </NewsView>
    </v-container>
</template>

<script>
    import {
        getReadingList,
	addToNotinterested,
	addToReadhistory,
        removeFromReadingList,
        addToFavourites
    } from './../api/api';
    import EmptyFeed from './sub-components/EmptyFeed.vue';
    import NewsCard from './sub-components/NewsCard.vue';
    import NewsView from './sub-components/NewsView.vue';

    export default {
        data() {
            return {
                readingList: [],
                selectedNews: {},
                selectedNewsIndex: -1,
                showNewsModal: false,
                loading: false
            };
        },
        components: {
            NewsCard,
            NewsView,
            EmptyFeed
        },
        watch: {
            '$route' () {
                this.fetchReadingList();
                this.readingList = [];
            }
        },
        mounted() {
            this.fetchReadingList();
            this.readingList = [];
        },
        methods: {
            fetchReadingList() {
                this.loading = true;
                getReadingList()
                    .then(data => {
                        if (data.error === undefined) {
                            if (data.success) {
                                let favourites = new Set(data.user.favourites);
				let notinteresteds = new Set(data.user.notinteresteds);

                                let readingList = data.readingList.map(element => {
                                    return {
                                        ...element,
                                        favourite: favourites.has(element.newsHash),
					notinterested: notinteresteds.has(element.newsHash)
                                    };
                                });
                                this.readingList.push(...readingList);
                            } else {
                                this.$emit('displayMessage', 'warning', data.message);
                            }
                        } else {
                            this.$emit('displayMessage', 'error', data.error);
                        }
                        this.loading = false;
                    });
            },
            saveNewsAsNotinterested(news, index) {
                addToNotinterested(news, news.hash)
                    .then(data => {
                        if (data.error === undefined) {
                            if (data.success) {
                                this.readingList[index].notinterested = true;

                                if (this.selectedNews.notinterested !== undefined) {
                                    this.selectedNews.notinterested = true;
                                }

                                this.$emit('displayMessage', 'success', data.message);
                            } else {
                                this.$emit('displayMessage', 'warning', data.message);
                            }
                        } else {
                            this.$emit('displayMessage', 'error', data.error);
                        }
                    });
            },
            saveNewsAsFavourite(news, index) {
                console.log(news);
                addToFavourites(news, news.newsHash)
                    .then(data => {
                        if (data.error === undefined) {
                            if (data.success) {
                                this.readingList[index].favourite = true;

                                if (this.selectedNews.favourite !== undefined) {
                                    this.selectedNews.favourite = true;
                                }

                                this.$emit('displayMessage', 'success', data.message);
                            } else {
                                this.$emit('displayMessage', 'warning', data.message);
                            }
                        } else {
                            this.$emit('displayMessage', 'error', data.error);
                        }
                    });
            },
            saveNewsAsReadhistory(news, index) {
                addToReadhistory(news, news.hash)
                    .then(data => {
                        if (data.error === undefined) {
                            if (data.success) {
                                this.readingList[index].readhistory = true;

                                if (this.selectedNews.readhistory !== undefined) {
                                    this.selectedNews.readhistory = true;
                                }

                                this.$emit('displayMessage', 'success', data.message);
                            } else {
                                this.$emit('displayMessage', 'warning', data.message);
                            }
                        } else {
                            this.$emit('displayMessage', 'error', data.error);
                        }
                    });
            },
            deleteNewsFromReadingList(item) {
                var hash = item.hash;
                removeFromReadingList(hash)
                    .then(data => {
                        if (data.error === undefined) {
                            if (data.success) {
                                this.readingList = this.readingList.filter(element => {
                                    return element.hash !== hash;
                                });
                                this.closeNewsModal();
                                this.$emit('displayMessage', 'success', data.message);
                            } else {
                                this.$emit('displayMessage', 'warning', data.message);
                            }
                        } else {
                            this.$emit('displayMessage', 'error', data.error);
                        }
                    });
            },
            viewNews(item, index) {
                this.showNewsModal = true;
                this.selectedNews = item;
                this.selectedNewsIndex = index;
            },
            closeNewsModal() {
                this.showNewsModal = false;
                this.selectedNews = {};
                this.selectedNewsIndex = -1;
            }
        }
    };
</script>
