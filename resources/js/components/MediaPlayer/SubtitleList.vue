<template>
    <v-container class="d-flex flex-column justify-center flex-nowrap">
        <!-- Media player info -->
        <v-alert
            v-if="!$props.subtitleLoading"
            class="media-player-subtitle-info mt-12 mb-6 rounded-lg"
            color="primary"
            type="info"
            border="left"
            dark
        >
            Choose a subtitle from the media currently being played on your Jellyfin server. 
            You can import them seamlessly into LinguaCafe for later reading and also 
            conveniently control your media player within LinguaCafe, allowing you to jump 
            to specific subtitles or stop the video.
        </v-alert>

        <!-- Subtitle list -->
        <v-card 
            outlined 
            id="subtitle-list" 
            class="rounded-lg px-4 pb-4" 
            :loading="subtitleListLoading || $props.subtitleLoading"
        >
            <!-- Subtitle list title-->
            <div id="subtitles-card-title" class="mt-4" v-if="!$props.subtitleLoading">
                Subtitles
                <v-btn 
                    id="refresh-button" 
                    rounded
                    color="primary" 
                    @click="loadSubtitleList" 
                    :disabled="subtitleListLoading" 
                    v-if="!$props.subtitleLoading"
                >
                    <v-icon class="mr-1">mdi-refresh</v-icon> Refresh
                </v-btn>
            </div>

            
            <!-- Subtitle list header -->
            <div class="regular-list-height subtitle header rounded-pill my-2" v-if="!$props.subtitleLoading">
                <div class="subtitle-language">Language</div>
                <div class="subtitle-user">User</div>
                <div class="subtitle-client">Client</div>
                <div class="subtitle-media">Media</div>
            </div>
            
            <!-- Subtitle list skeleton loader -->
            <template v-if="subtitleListLoading">
                <v-skeleton-loader
                    v-for="index in 3"
                    :key="index"
                    class="regular-list-height d-block skeleton rounded-pill my-2"
                    type="image"
                ></v-skeleton-loader>
            </template>
            
            <!-- Subtitle list body -->
            <template v-for="(session, sessionIndex) in sessions" v-if="!subtitleListLoading && !$props.subtitleLoading">
                <div 
                    class="regular-list-height subtitle rounded-pill my-2" 
                    @click="selectSubtitle(sessionIndex, subtitleIndex)"
                    v-for="(subtitle, subtitleIndex) in session.subtitles"
                    :key="sessionIndex + '-' + subtitleIndex"
                >
                    <div class="subtitle-language">
                        <v-img 
                            class="border mx-auto" 
                            :src="'/images/flags/' + subtitle.language" 
                            max-width="43" 
                            height="28"
                        ></v-img> 
                    </div>
                    <div class="subtitle-user">{{ session.userName }}</div>
                    <div class="subtitle-client">{{ session.client }}</div>
                    <div class="subtitle-media">{{ session.seriesName }} S{{ ('0' + session.seriesSeason).slice(-2) }}E{{ ('0' + session.seriesEpisode).slice(-2) }} - {{ session.title }}</div>
                </div>
            </template>

            <!-- Subtitle processing title -->
            <div id="subtitles-card-title" class="mt-4 processing" v-if="$props.subtitleLoading">
                Subtitle processing
            </div>

            <!-- Subtitle processing info -->
            <div class="flex justify-space-around" v-if="$props.subtitleLoading">
                <v-alert
                    class="media-player-subtitle-info my-6 rounded-lg"
                    color="primary"
                    border="left"
                    dark
                    icon="mdi-progress-clock"
                >
                    Your selected subtitle is being processed. This can take 10 to 30 seconds. 
                    Once processed, it will be cached for quicker loading in the future.
                </v-alert>
            </div>
        </v-card>
    </v-container>
</template>


<script>
export default {
    data: function () {
        return {
            subtitleListLoading: false,
            sessions: []
        }
    },
    props: {
        subtitleLoading: Boolean
    },
    mounted() {
        this.loadSubtitleList();
    },
    methods: {
        loadSubtitleList: function () {
            this.subtitleListLoading = true;
            this.sessions = [];
            axios.get('/jellyfin/subtitles').then(async (result) => {
                this.sessions = result.data;
                this.subtitleListLoading = false;
            });
        },
        selectSubtitle: function(selectedSession, selectedSubtitle) {
            this.$emit('subtitle-change', {
                subtitle: this.sessions[selectedSession].subtitles[selectedSubtitle].text,
                language: this.sessions[selectedSession].subtitles[selectedSubtitle].language,
                client: this.sessions[selectedSession].client,
                userName: this.sessions[selectedSession].userName,
                userId: this.sessions[selectedSession].userId,
                title: this.sessions[selectedSession].title,
                seriesName: this.sessions[selectedSession].seriesName,
                seriesEpisode: this.sessions[selectedSession].seriesEpisode,
                seriesSeason: this.sessions[selectedSession].seriesSeason,
                nowPlayingItemId: this.sessions[selectedSession].nowPlayingItemId,
                runTimeTicks: this.sessions[selectedSession].runTimeTicks,
                mediaSourceId: this.sessions[selectedSession].mediaSourceId,
                sessionId: this.sessions[selectedSession].sessionId
            });
        }
    }
}
</script>
