<script setup>
const route = useRoute();
const state = reactive({
  data: null,
  isLike: false,
  content:[],
})

await fetchApi.get(apiPath.public.News.get.byId(route.params.id))
  .then((res) => {
    state.data = res.data.value.data
    publicbreadcrumbstore().setBreadCrumbs([
    {
      title: 'خانه',
      disabled: false,
      to: '/',
    },
    {
      title: 'خبر ها',
      disabled: false,
      to: '/news',
    },
    {
      title: state.data?.title,
      disabled: true,
      to: `/news/${state.data?.id}`,
    }])
  }).catch(error => {
    common.showError(error?.messages)
  })

useHead({
  title: `${state.data?.title}`,
  meta:[{name:'description',content:`${state.data?.summary}`}],
  meta:[{name:'keywords',content:`${state.data?.keywordList}`}]
})
//#endregion

//#region LIKE
const putLike = async (id) => {
state.content = JSON.parse(localStorage.getItem("content")) ?? []

  if (!state.content.includes(id))
    await axiosApi().put(apiPath.public.Content.put.like(id))
      .then((res) => {
        if (res) {
          state.data.like = res.data
          state.content.push(id)
          localStorage.setItem("content", JSON.stringify(state.content))
        }
      })
      .catch((error) => {
        common.showError(error?.data?.messages)
      })
}
//#endregion

//#region DISLIKE
const putDislike = async (id) => {
  state.content = JSON.parse(localStorage.getItem("content")) ?? []

  if (!state.content.includes(id))
  await axiosApi().put(apiPath.public.Content.put.dislike(id))
    .then((res) => {
      if (res) {
        state.data.disLike = res.data
        state.content.push(id)
          localStorage.setItem("content", JSON.stringify(state.content))
      }
    })
    .catch((error) => {
      common.showError(error?.data?.messages)
    })
}
//#endregion

</script>
<template>
  <v-container>
    <v-row>
      <v-col cols="12" :md="8" lg="9">
        <v-card class="single-content rounded-xl mb-5">
          <v-card-title>
            <div class="mb-8">
              <h4>{{ state.data?.title }}</h4>
            </div>
            <div>
              <v-row>
                <v-col class="d-flex">
                  <div class="text-primary ml-5">
                    <v-icon size="16" class="ml-2">mdi-account-outline</v-icon>
                    <small>{{ `${state.data?.author?.firstName} ${state.data?.author?.lastName}` }}</small>
                  </div>
                  <span class="text-blue ml-5">
                    <v-icon size="16" class="ml-2">mdi-calendar-month-outline</v-icon>
                    <small>{{ dateConverter.convertToJalali(state.data?.createdOn) }}</small>
                  </span>
                  <span class="ml-9">
                    <v-icon size="16" class="ml-2">mdi-eye-outline</v-icon>
                    <small>{{ state.data?.visitedCount }}</small>
                  </span>
                </v-col>
                <v-col class="text-end">
                  <v-btn color="success" class="border-none" @click="putLike(state.data?.id)">
                    <span>
                      <v-icon size="16" class="ml-2">mdi-thumb-up-outline</v-icon>
                      <small>{{ state.data?.like }}</small>
                    </span>
                  </v-btn>
                  <v-btn color="red" class="border-none" @click="putDislike(state.data?.id)">
                    <span>
                      <v-icon size="16" class="ml-2">mdi-thumb-down-outline</v-icon>
                      <small>{{ state.data?.disLike }}</small>
                    </span>
                  </v-btn>
                  <a href="#Comment">
                    <v-btn color="success" class="border-none ml-5">
                      <v-icon size="16">mdi-chat-outline</v-icon>
                      <small>{{ state.data?.comment }}</small>
                    </v-btn>
                  </a>
                </v-col>
              </v-row>
            </div>
          </v-card-title>
          <v-divider></v-divider>
          <v-card-item>
            <BaseImage :src="state.data?.imageIds" max-width="400" class="mx-auto" :alt="state.data?.title"/>
          </v-card-item>
          <v-card-text>
            <div v-html="state.data?.body">
            </div>
          </v-card-text>
        </v-card>
        <NewsCommentComp />
      </v-col>
      <v-col cols="12" :md="4" lg="3">
        <NewsAsideLatestItemsCompname="News"/>
      </v-col>
    </v-row>

  </v-container>
</template>