<template>
    <div id="admin-api-settings" v-if="settings">
        <!-- DeepL settings -->
        <div class="subheader mt-4">DeepL</div>
        <v-card outlined class="rounded-lg pa-4 pt-0" :loading="characterLimitLoading">
            <v-card-text id="deepl-card-text">
                <!-- DeepL cache -->
                <label class="font-weight-bold" v-if="characterLimitStatus !== 'error'">
                    DeepL Cache
                </label>
                
                <div v-if="characterLimitLoading">
                    <v-skeleton-loader
                        id="skeleton-cached-translations"
                        class="regular-list-height d-block skeleton rounded-pill mt-1"
                        type="image"
                    ></v-skeleton-loader>
                </div>
                
                <div v-if="!characterLimitLoading && characterLimitStatus !== 'error'">
                    {{ formatNumber(cachedDeeplTranslations).replace('&nbsp;', '') }} cached translations.
                </div>

                <!-- DeepL API key input -->
                <label class="font-weight-bold mt-4">DeepL API key</label>
                <v-text-field 
                    v-model="settings.deeplApiKey"
                    class="mb-4"
                    hide-details
                    filled
                    dense
                    rounded
                    placeholder="DeepL API key"
                    :disabled="saving || characterLimitLoading"
                ></v-text-field>

                <!-- DeepL API usage -->
                <label 
                    v-if="characterLimitStatus !== 'error'"
                    class="font-weight-bold mt-4" 
                >
                    DeepL character usage
                </label>

                <v-card id="deepl-api-card" class="rounded-lg pa-6" elevation="0" v-if="characterLimitStatus !== 'error'">
                    <!-- DeepL API usage skeleton -->
                    <v-card-text class="pa-0" v-if="characterLimitLoading">
                        <v-skeleton-loader
                            id="skeleton-used-characters"
                            class="regular-list-height d-block skeleton rounded-pill mt-2 mb-3"
                            type="image"
                        ></v-skeleton-loader>

                        <v-skeleton-loader
                            id="skeleton-max-characters"
                            class="regular-list-height d-block skeleton rounded-pill"
                            type="image"
                        ></v-skeleton-loader>

                        <v-skeleton-loader
                            id="skeleton-progress-bar"
                            class="regular-list-height d-block skeleton rounded-pill mt-6 mb-2"
                            type="image"
                        ></v-skeleton-loader>

                         <v-skeleton-loader
                            id="skeleton-characters-left"
                            class="regular-list-height d-block skeleton rounded-pill mt-1"
                            type="image"
                        ></v-skeleton-loader>
                    </v-card-text>

                    <!-- DeepL API usage loaded -->
                    <v-card-text class="pa-0" v-if="!characterLimitLoading">
                        <div class="font-weight-bold mt-2 mb-3" style="font-size: 36px;">
                            {{ formatNumber(characterUsed).replace('&nbsp;', '') }}
                        </div>
                        
                        <div>Out of max.{{ formatNumber(characterLimit) }} characters</div>
                        
                        <v-progress-linear
                            color="primary"
                            height="36"
                            :value="this.characterUsed / this.characterLimit * 100"
                            class="rounded-pill mt-6 mb-2"
                        >
                            <strong></strong>
                        </v-progress-linear>
                        {{ formatNumber(characterLimit - characterUsed).replace('&nbsp;', '') }} characters left
                    </v-card-text>
                </v-card>

                <!-- DeepL API key error message -->
                <v-alert
                    v-if="!saving && characterLimitStatus == 'error'"
                    class="rounded-lg mt-2"
                    color="error"
                    type="error"
                    border="left"
                    dark
                >
                    DeepL API call failed. Please make sure that your API key is valid and DeepL services are online.
                </v-alert>
            </v-card-text>
        </v-card>

        <!-- Anki connect settings -->
        <div class="subheader subheader-margin-top">Anki</div>
        <v-card outlined class="rounded-lg pa-4 pt-0">
            <v-card-text id="jellyfin-card-text">
                <label class="font-weight-bold">Anki-connect host</label>
                <v-text-field 
                    v-model="settings.ankiConnectHost"
                    hide-details
                    filled
                    dense
                    rounded
                    placeholder="Anki-connect host"
                    :disabled="saving || characterLimitLoading"
                ></v-text-field>

                <label class="font-weight-bold mt-4 mb-0">Auto add cards while reading</label>
                <v-switch
                    v-model="settings.ankiAutoAddCards"
                    class="mt-0"
                    color="primary"
                    hide-hints
                    dense
                    label="Auto add cards"
                ></v-switch>

                <label class="font-weight-bold mt-1 mb-0">Update existing cards</label>
                <v-switch
                    v-model="settings.ankiUpdateCards"
                    class="mt-0"
                    color="primary"
                    hide-hints
                    dense
                    label="Update existing cards"
                ></v-switch>

                <label class="font-weight-bold mt-1 mb-0">Show notifications</label>
                <v-switch
                    v-model="settings.ankiShowNotifications"
                    class="mt-0"
                    color="primary"
                    hide-hints
                    dense
                    label="Show notifications"
                ></v-switch>
            </v-card-text>
        </v-card>

        <!-- Jellyfin settings -->
        <div class="subheader subheader-margin-top">Jellyfin</div>
        <v-card outlined class="rounded-lg pa-4 pt-0">
            <v-card-text id="jellyfin-card-text">
                <label class="font-weight-bold">Jellyfin host address</label>
                <v-text-field 
                    v-model="settings.jellyfinHost"
                    hide-details
                    filled
                    dense
                    rounded
                    placeholder="Jellyfin host address"
                    :disabled="saving || characterLimitLoading"
                ></v-text-field>

                <label class="font-weight-bold mt-4">Jellyfin API key</label>
                <v-text-field 
                    v-model="settings.jellyfinApiKey"
                    hide-details
                    filled
                    dense
                    rounded
                    placeholder="Jellyfin API key"
                    :disabled="saving || characterLimitLoading"
                ></v-text-field>
            </v-card-text>
        </v-card>

        <!-- Save result alerts -->
        <v-alert
            v-if="!saving && saveStatus !== '' && saveStatus !== 'success'"
            class="rounded-lg my-3"
            color="error"
            type="error"
            border="left"
            dark
        >
            An error has occurred while saving API settings.
        </v-alert>

        <!-- Save button -->
        <div class="d-flex">
            <v-spacer />
            <v-btn 
                rounded 
                :class="{'my-2': saving || saveStatus == '' || saveStatus == 'success'}"
                color="primary"
                @click="saveSettings"
                :disabled="saving || characterLimitLoading"
                :loading="saving || characterLimitLoading"
            >
                Save settings
            </v-btn>
        </div>
    </div>
</template>

<script>
    import {formatNumber} from './../../helper.js';
    export default {
        data: function() {
            return {
                settings: null,
                saving: false,
                saveStatus: '',
                characterLimitLoading: true,
                characterLimitStatus: '',
                characterUsed: 0,
                characterLimit: 0,
                cachedDeeplTranslations: 0,
            }
        },
        props: {
            language: String
        },
        mounted() {
            this.loadSettings();
        },
        methods: {
            loadDeeplCharacterLimits() {
                axios.get('/dictionaries/deepl/get-usage').then((result) => {
                    this.characterLimitLoading = false;
                    if (result.data == 'error') {
                        this.characterLimitStatus = 'error';
                    } else {
                        this.characterLimitStatus = 'success';
                        this.cachedDeeplTranslations = result.data.cachedDeeplTranslations;
                        this.characterUsed = result.data.limits.character.count;
                        this.characterLimit = result.data.limits.character.limit;
                    }
                });
            },
            loadSettings() {
                axios.post('/settings/get-by-name', {
                    'settingNames': [
                        'deeplApiKey',
                        'jellyfinHost',
                        'jellyfinApiKey',
                        'ankiConnectHost',
                        'ankiAutoAddCards',
                        'ankiUpdateCards',
                        'ankiShowNotifications'
                    ]
                }).then((result) => {
                    this.settings = result.data;
                    this.loadDeeplCharacterLimits();
                });
            },
            saveSettings() {
                this.saving = true;
                
                this.characterLimitLoading = true;
                this.characterUsed = 0;
                this.characterLimit = 0;
                this.characterLimitStatus = '';

                axios.post('/settings/save', {
                    'settings': {
                        'deeplApiKey': this.settings.deeplApiKey,
                        'jellyfinHost': this.settings.jellyfinHost,
                        'jellyfinApiKey': this.settings.jellyfinApiKey,
                        'ankiConnectHost': this.settings.ankiConnectHost,
                        'ankiAutoAddCards': this.settings.ankiAutoAddCards,
                        'ankiUpdateCards': this.settings.ankiUpdateCards,
                        'ankiShowNotifications': this.settings.ankiShowNotifications
                    }
                }).then((result) => {
                    this.saving = false;
                    this.saveStatus = result.data;
                    this.loadDeeplCharacterLimits();
                });
            },
            formatNumber: formatNumber,
        }
    }
</script>
