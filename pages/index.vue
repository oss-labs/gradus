<template>
  <!-- Header -->
  <CoreHeader />
  <!-- Header -->

  <v-main>
    <CoreAppBanner v-if="config.config.banner.status" />
    <CoreHeroCard />

    <v-container
      fluid
      style="
        border-bottom: 1px solid #dee5f1 !important;
        border-top: 1px solid #dee5f1 !important;
      "
    >
      <v-row justify="center" align="center">
        <v-col md="11">
          <v-container fluid class="pa-0">
            <v-row justify="space-between">
              <v-col md="3" lg="3">
                <v-text-field
                  :prepend-icon="prependIconSearch"
                  v-model="search"
                  hide-details="auto"
                  rounded
                  variant="outlined"
                  clearable
                  bg-color="#E8F0FE"
                  base-color="#DEE5F1"
                  append-inner-icon="mdi-magnify"
                  density="compact"
                  @click:clear="resetData()"
                  @input="searchData($event)"
                  label="Search for any topic"
                ></v-text-field>
              </v-col>
              <v-col md="3" lg="3">
                <v-select
                  v-model="filter"
                  :prepend-icon="prependIconFilter"
                  closable-chips
                  label="Choose Topic"
                  density="compact"
                  variant="outlined"
                  clearable
                  rounded
                  multiple
                  bg-color="#E8F0FE"
                  base-color="#DEE5F1"
                  dense
                  :items="topics"
                  @click:clear="resetData()"
                  @update:modelValue="filterData($event)"
                  @input="filterData($event)"
                  hide-details
                >
                  <template v-slot:selection="{ item, index }">
                    <v-chip size="small" v-if="index < 1">
                      <span>{{ item.title }}</span>
                    </v-chip>
                    <span
                      v-if="index === 1"
                      class="text-grey text-caption align-self-center"
                    >
                      (+{{ filter.length - 1 }} others)
                    </span>
                  </template>
                </v-select>
              </v-col>
            </v-row>
          </v-container>
        </v-col>
      </v-row>
    </v-container>

    <v-container fluid class="my-0 py-0">
      <!-- Cards -->
      <v-row
        justify="center"
        align="center"
        class="px-0 mx-0 mt-5"
        v-if="res != null && Object.keys(res).length"
      >
        <v-col md="11" class="px-0">
          <v-container fluid class="px-0">
            <v-row>
              <v-col
                md="3"
                lg="3"
                sm="4"
                cols="12"
                v-for="(article, index) in res"
                :key="index"
              >
                <CoreLabCard :item="article" />
              </v-col>
            </v-row>
          </v-container>
        </v-col>
      </v-row>
      <!-- Cards -->

      <!-- No Data -->
      <v-row justify="center" align="center" class="mx-0 mt-3" v-else>
        <v-col md="11" class="px-0">
          <p>
            <v-icon class="mt-n1 mr-2">mdi-alert-octagram-outline</v-icon>No
            Data Found!
          </p>
        </v-col>
      </v-row>
      <!-- No Data -->
    </v-container>
  </v-main>

  <!-- Footer -->
  <CoreAppFooter />
  <!-- Footer -->
</template>

<script setup>
import { config } from "../assets/config.js";
import topicFilters from "../assets/topic_filter.json";

// import { useDebounce } from "@vueuse/core";

useSeoMeta({
  contentType: "text/html; charset=utf-8",
  title: config.name,
  description: config.dsec,
  keywords: config.seo.keywords,
  author: config.seo.author,
  creator: "Gradus",
  viewport: "width=device-width, initial-scale=1.0",
  ogTitle: config.name,
  ogDescription: config.dsec,
  ogImage: `${config.hostUrl}${config.thumbnail}?auto=format&fit=crop&frame=1&h=512&w=1024`,
  ogUrl: config.hostUrl,
  ogType: "website",
  twitterTitle: config.name,
  twitterDescription: config.dsec,
  twitterImage: `${config.hostUrl}${config.thumbnail}?auto=format&fit=crop&frame=1&h=512&w=1024`,
  twitterCard: "summary_large_image",
});

const router = useRouter();
// const filter = ref([]);
const topics = ref(topicFilters);
const search = ref("");
// const debouncedSearch = useDebounce(search, 500);
const res = ref({});
const actualRes = ref({});
const query = ref({});
const prependIconFilter = ref("mdi-filter-outline");
const prependIconSearch = ref("mdi-card-search-outline");

const appLoading = useAppLoading();

onMounted(() => {
  if (window.innerWidth < 600) {
    prependIconFilter.value = "";
    prependIconSearch.value = "";
  } else {
    prependIconFilter.value = "mdi-filter-outline";
    prependIconSearch.value = "mdi-card-search-outline";
  }
});

function getFilterFromUrl() {
  const urlParams = new URLSearchParams(router.currentRoute.value.query);
  const filter = urlParams.get("filter");
  return filter ? filter.split(",") : [];
}

const filter = ref(getFilterFromUrl());

function setFilterToUrl(filter) {
  const urlParams = new URLSearchParams(router.currentRoute.value.query);

  if (filter.length === 0) {
    urlParams.delete("filter");
  } else {
    urlParams.set("filter", filter.join(","));
  }

  router.push({ query: Object.fromEntries(urlParams.entries()) });
}

const fetchData = async () => {
  query.value = { draft: false };
  const { data } = await useAsyncData("allData", () =>
    queryContent().where(query.value).sort({ date: -1 }).find()
  );
  res.value = data._rawValue;
  actualRes.value = data._rawValue;
  appLoading.value = false;
};

const filterData = (a) => {
  if (a.length === 0) {
    resetData();
    return;
  }
  const upperCaseTags = a.map((tag) => tag.toUpperCase());
  setFilterToUrl(upperCaseTags);

  res.value = actualRes.value.filter((obj) =>
    obj.tags.some((objTag) => upperCaseTags.includes(objTag.toUpperCase()))
  );
};

const resetData = () => {
  res.value = actualRes.value;
  setFilterToUrl([]);
};

const searchData = () => {
  res.value = actualRes.value.filter((obj) =>
    Object.values(obj).some(
      (value) => typeof value === "string" && value.includes(search.value)
    )
  );
};

fetchData();
</script>

<style></style>
