<template>
    <v-flex xs12 sm6 md4 lg3 d-flex style="margin: 7px 0">
        <v-card raised class="flex-display">
            <v-card-media
                class="white--text image-zoom"
                height="200px"
                :src="item.image"
            >
                <v-container fill-height fluid>
                    <v-layout fill-height>
                        <v-flex xs12 align-end flexbox>
                            <span class="img-cont">{{ item.title | decodeHTML }}</span>
                        </v-flex>
                    </v-layout>
                </v-container>
            </v-card-media>
            <v-card-title class="flex-content" style="background: white">
                <div class="truncated-text-display">
                    <span style="color: #5a5e66">{{ item.description | removeHTML | truncate }}</span>
                </div>
            </v-card-title>
            <v-card-actions class="flex-footer" style="background: white">
                <v-spacer></v-spacer>
                <div class="body-2" style="margin: 0 14px">{{ item.date | ago | titleCase }}</div>
            </v-card-actions>
            <v-card-actions class="flex-footer">
                
		<slot name="slot_0"></slot>
                <slot name="slot_1"></slot>
                <slot name="slot_4"></slot>		
                <slot name="slot_2"></slot>
                <slot name="slot_3"></slot>

            </v-card-actions>
        </v-card>
    </v-flex>
</template>


<script>
    export default {
        props: {
            item: {
                type: Object,
                required: true
            },
            viewNews: {
                type: Function,
                required: true
            },
            index: {
                type: Number,
                required: true
            },
            className: {
                type: String,
                required: true
            }
        }
    }
</script>
