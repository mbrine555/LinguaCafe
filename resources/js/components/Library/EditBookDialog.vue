<template>
    <v-dialog v-model="value" persistent width="800px">
        <v-card id="edit-book-dialog" class="rounded-lg">
            <!-- Card title -->
            <v-card-title>
                <!-- Edit book title -->
                <template v-if="$props.bookId !== -1">
                    <v-icon class="mr-2">mdi-folder-edit</v-icon>Edit book
                </template>

                <!-- New book title -->
                <template v-if="$props.bookId == -1">
                    <v-icon class="mr-2">mdi-folder-plus</v-icon>Add book
                </template>

                <v-spacer />
                <v-btn icon @click="close">
                    <v-icon>mdi-close</v-icon>
                </v-btn>
            </v-card-title>
            
            <!-- Form -->
            <v-card-text>
                <v-form ref="bookForm">
                    <label class="font-weight-bold">Book name</label>
                    <v-text-field 
                        v-model="name"
                        filled
                        dense
                        rounded
                        placeholder="Name"
                        :rules="[rules.name]"
                        maxlength="128"
                        @change="validateForm"
                    ></v-text-field>
                    
                    
                    <label class="font-weight-bold mt-2">Book cover image</label><br>
                    <template v-if="editImage">
                        <v-file-input
                            v-model="image"
                            filled
                            dense
                            rounded
                            ref="image"
                            :rules="[rules.image]"
                            accept="image/*"
                            placeholder="Cover image"
                            prepend-icon="mdi-image"
                            @change="validateForm"
                        ></v-file-input>
                    </template>
                    <template v-if="!editImage">
                        <div id="image-upload-box" class="d-flex">
                            <div id="image-box" class="d-flex align-center">
                                <img 
                                    class="cover-image rounded-xl"
                                    :src="'/images/book_images/' + $props.bookCover"
                                    width="100px"
                                ></img>
                            </div>
                            <div
                                id="edit-book-upload-image-button" 
                                class="rounded-xl bg-foreground-base"
                                @click="uploadImageButton"
                            >
                                
                                <h4><v-icon large>mdi-file-upload</v-icon> Change image</h4>
                            </div>
                        </div>
                    </template>
                    
                    <v-alert
                        class="my-3" 
                        border="left"
                        type="error"
                        v-if="saveResult == 'error'"
                    >
                        An error has occurred while saving.
                    </v-alert>

                    <v-alert
                        class="my-3" 
                        border="left"
                        type="success"
                        v-if="saveResult == 'success'"
                    >
                        Chapter has been saved successfully.
                    </v-alert>
                </v-form>
            </v-card-text>

            <!-- Action buttons -->
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn rounded text @click="close">Cancel</v-btn>

                <v-btn 
                    rounded 
                    depressed
                    color="primary" 
                    @click="save"
                    :disabled="!isFormValid || saving || saveResult == 'success'"
                    :loading="saving"
                >
                    Save
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
    export default {
        data: function() {
            return {
                isFormValid: false,
                saving: false,
                saveResult: '',
                name: this.$props.bookName,
                image: null,
                editImage: false,

                rules: {
                    name: (value) => {
                        if (value === null || !value.length) {
                            return 'You must type in a name.'
                        }

                        return true;
                    },
                    image: (value) => {
                        if (value === null) {
                            return 'You must select a cover image.'
                        }

                        return true;
                    },
                },
            }
        },
        props: {
            value : Boolean,
            bookId: Number,
            bookName: String,
            bookCover: String
        },
        emits: ['input'],
        mounted() {
            this.validateForm();
        },
        methods: {
            uploadImageButton() {
                this.editImage = true;
                this.isFormValid = false;
                this.$nextTick(() => {
                    this.$refs.image.$refs.input.click();
                });
            },
            validateForm() {
                this.isFormValid = this.$refs.bookForm.validate();
            },
            save() {
                if (!this.$refs.bookForm.validate()) {
                    this.isFormValid = false;
                    return;
                }

                this.saving = true;
                
                var form = new FormData();
                form.set('bookId',this.$props.bookId);
                form.set('bookName',this.name);
                if (this.editImage) {
                    form.set('bookCover', this.image);
                }

                axios.post('/book/save', form).then((response) => {
                    this.saving = false;
                    if (response.data == 'success') {
                        this.saveResult = 'success';

                        setTimeout(() => {
                            this.$emit('book-saved');
                            this.close();
                        }, 750);
                    } else {
                        this.saveResult = 'error';
                    }
                });
            },
            close() {
                this.$emit('input', false);
            }
        }
    }
</script>
