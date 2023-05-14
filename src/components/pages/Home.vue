<template>
    <div class="d-flex align-center flex-column">
  
      <template v-if="loading">
        <v-card>
          <v-skeleton
            class="mx-auto"
            width="85%"
            type="card"
          ></v-skeleton>
          <v-skeleton-loader
            class="mx-auto"
            width="55%"
            type="card"
          ></v-skeleton-loader>
          <v-skeleton-loader
            class="mx-auto"
            width="70%"
            type="card"
          ></v-skeleton-loader>
        </v-card>
      </template>
      <template v-else>
        <!-- <v-button-group>
          <v-button @click="Next(-1)" v-if="this.meta && this.meta.currentPage !==1">Previous</v-button>
          {{ meta.currentPage }} / {{ meta.totalPages }} ( {{ meta.totalItems }})
          <v-button @click="Next(1)" v-if="this.meta && this.meta.totalPages !== this.meta.currentPage">Next</v-button>
        </v-button-group> -->
        <v-pagination
          v-model="meta.currentPage"
          :length="meta.totalPages"
          rounded="circle"
        ></v-pagination>
        <template v-if="items.length > 0">
          <v-card v-for="item in items" :key="item.id" @click="View(item)">
            <v-card-img img-top
                        tag="article"
                        style="max-width: 20rem;"
                        class="mv-2" :src="item.imageData" ></v-card-img>
            <v-card-body>
              <v-card-text>
                {{ JSON.parse(item.jsonResult).text }}
              </v-card-text>
            </v-card-body>
          </v-card>
        </template>
        <template v-else>
          <v-card
            width="400"
            title="There is no information"
            subtitle="Please uplaod some DL"
          ></v-card>
        </template>
  
      </template>
    </div>
</template>

<script setup>
  import { ref, onMounted } from "vue"
  import { useRouter } from "vue-router"
  import config from '../../config'

  const router = useRouter()
  const loading = ref(true)
  const userMessage = ref("")
  const meta = ref({
    totalItems: 0,
    itemCount: 0,
    itemsPerPage: 0,
    totalPages: 0,
    currentPage: 0,
  })
  const items = ref([])

  const View = item => {
    router.push(`/view/${item.id}`)
  }

  const Next = async i => {
    if (meta && meta.currentPage + i >= 0 && meta.currentPage + i <= meta.totalPages)
      await ListWrapper(meta.currentPage + i);
  }

  const ListWrapper = async i => {
    loading.value = true;
  
    try {
      const result = await List(i, 25);
      meta.value = result.meta;
      item.value = result.items;

      console.log(result);
    } catch (e) {
      loading.value = false;
      userMessage.value = e.message;
    }
    // loading.value = false;
  }

  const List = async (page, limit) => {
    const result = await (fetch(`${ config.SERVER_URL }/api/upload/list`, {
      method: "POST",
      body: JSON.stringify({
        limit: limit,
        page: page
  
      })
    }));
    if (result.ok)
      return await result.json();
  }

  onMounted(async () => {
    await ListWrapper(1);
  })
  </script>
  