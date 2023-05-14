<template>
  <v-container>
    <!-- <user-messages :error="error" :user-message="userMessage" /> -->
    <v-card v-if="loading">
      <v-skeleton animation="wave" width="85%"></v-skeleton>
      <v-skeleton animation="wave" width="55%"></v-skeleton>
      <v-skeleton animation="wave" width="70%"></v-skeleton>
    </v-card>
    <v-row justify="center" v-else>
      <v-col cols="6">
        <v-form>
          <v-text-field label="Processor Id" v-model="pId" />
          <v-file-input
            label="Drag and drop or select a file. It will be auto processed. You must put in an Processor Id to enable this input."
            placeholder=""
            drop-placeholder="Drag and drop a file"
            v-model="file"
            :disabled="hasId()"
          />
          <v-btn right @click="FileSelected">Read File</v-btn>
        </v-form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref } from 'vue'
import { Upload } from "./upload.api"

const pId = ref("4137328724cc9c6a");
const error = ref(false);
const userMessage = ref("");
const loading = ref(false);
const file = ref(null);

const hasId = () => {
  return pId.value.trim().length === 0;
}

const FileSelected = async () => {
  loading.value = true;
  const reader = new FileReader();
  reader.readAsDataURL(file.value[0]);
  reader.onload = async () => {
    let resp;
    try {
      resp = await Upload(reader.result, pId.value);
    } catch (e) {
      error.value = true;
      userMessage.value = e.message;
      loading.value= false;
      return;
    }
    if (resp && resp.id) {
      await this.$router.push(`/view/${resp.id}`);
      loading.value= false;
      return;
    }
    console.log("Could not read file");
    error.value = true;
    userMessage.value = "Server could not get file";
    loading.value= false;
  };
}
</script>