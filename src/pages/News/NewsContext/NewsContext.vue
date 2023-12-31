<script setup lang="ts">
import Category from '@/components/Category/BaseCategory.vue';
import Pagination from '@/components/Pagination/BasePagination.vue';
import router from '@/router/index';
import { ref, watch, onMounted, onUnmounted, type PropType } from 'vue';
import useAxios, { type DataResponse } from '@/hooks/useAxios';
import LoadingComponentVue from '@/components/LoadingComponent/LoadingComponent.vue';

//GET DATA
interface ItemRS {
  id: string;
  title: string;
  img: string;
  slug: string;
  summary: string;
  detail: string;
  detailMobile: string;
  highlight: number;
  createAt: string;
  tags: [
    {
      id: string;
      name: string;
      slug: string;
      createAt: string;
    }
  ];
}

//Props
const content = defineProps({
  listItem: {
    type: Array as PropType<ItemRS[]>,
    required: false
  },
  prsPage: {
    type: Number,
    require: false
  },
  totalPage: {
    type: Number,
    required: false
  },
  popularStatus: {
    type: Boolean,
    require: false
  },
  path: {
    type: String,
    require: false
  },
  loading: {
    type: Boolean
  }
});

const emits = defineEmits<{
  // eslint-disable-next-line no-unused-vars
  (e: 'current-page', numberPage: number): void;
}>();

//Properties
const deps = ref([]);
const currentPage = ref(content && content.prsPage ? content.prsPage + 1 : 1);
const listData = ref<ItemRS[]>();
const addData = ref<ItemRS[]>();
const filterTags = ref();
const popular = ref();
const isLoading = ref(false);
const isLoading1 = ref(false);
const pageSize = ref(8);
const totalNews = ref();
const isDesktop = ref(true);

watch(content, () => {
  listData.value = content.listItem;
  popular.value = content.popularStatus;
  filterTags.value = content.path;
  isLoading.value = content.loading;
  totalNews.value = content.totalPage;
});

const displayNews = ref(listData);

const checkScreenSize = () => {
  if (window.innerWidth < 739) {
    isDesktop.value = false;
  } else {
    isDesktop.value = true;
  }
};

const scrollToTop = (top: number) => {
  window.scrollTo({
    top: top,
    behavior: 'smooth'
  });
};

const handlePageChange = (page: number) => {
  currentPage.value = page - 1;
  emits('current-page', currentPage.value);
  if (window.innerWidth < 739) {
    isDesktop.value = false;
    scrollToTop(0);
  } else {
    scrollToTop(550);
  }
};

//Button Readmore
const readMore = () => {
  isLoading1.value = true;
  currentPage.value++;
  const readMore = useAxios<DataResponse>(
    'get',
    // eslint-disable-next-line max-len
    `/news?${filterTags.value}&sort=desc&page=${currentPage.value}&pageSize=8&popular=${popular.value}`,
    {},
    {},
    deps.value
  );
  watch(readMore.response, () => {
    addData.value = readMore.response.value?.data?.data;
    addData.value?.forEach((item) => {
      displayNews.value?.push(item);
    });
  });

  watch(readMore.isLoading, () => {
    isLoading1.value = readMore.isLoading.value;
  });
};

const formatDate = (timestamp: string) => {
  const date = new Date(timestamp);
  const day = date.getDate().toString().padStart(2, '0');
  const month = (date.getMonth() + 1).toString().padStart(2, '0');
  const year = date.getFullYear().toString();
  const hours = date.getHours().toString().padStart(2, '0');
  const minutes = date.getMinutes().toString().padStart(2, '0');

  return `${day}/${month}/${year}, ${hours}:${minutes}`;
};

//Go to detail Page
const linkDetail = (slug: string) => {
  router.push(`/tintuc/${slug}`);
};

onMounted(() => {
  checkScreenSize();
  window.addEventListener('resize', checkScreenSize);
});

onUnmounted(() => {
  window.removeEventListener('resize', checkScreenSize);
});
</script>
<template>
  <div :class="$style.news__context">
    <div :class="$style['news__context-left']">
      <LoadingComponentVue v-if="isLoading" />
      <div
        :class="$style['news__context-item']"
        v-for="(item, index) in displayNews"
        :key="index"
        @click="linkDetail(item.slug)"
      >
        <div :class="$style['news__item-left']">
          <img :src="item.img" alt="BGItem" />
        </div>
        <div :class="$style['news__item-right']">
          <div :class="$style['news__item-date']">
            <p>{{ formatDate(item.createAt) }}</p>
            <span></span>
          </div>
          <h4>{{ item.title }}</h4>
          <div :class="$style['news__item-summary']">
            <span v-html="item.summary"></span>
          </div>
          <div :class="$style['news__item-footage']">
            <div :class="$style['news__footage-line']"></div>
            <span
              style="text-transform: uppercase; margin-right: 2rem"
              v-for="(subitem, index) in item.tags"
              :key="index"
              >{{ subitem.name }}</span
            >
            <div :class="$style['news__footage-line--mb']"></div>
          </div>
        </div>
      </div>

      <pagination
        v-if="content.listItem"
        :class="$style['news__context-left-pagination']"
        style="margin-top: 50px"
        :total="totalNews"
        :current-page="currentPage"
        :page-size="pageSize"
        @current-change="handlePageChange"
      />
      <div :class="$style['news__context-left-loading']">
        <ul :class="$style['loader-list']" v-if="isLoading1">
          <li>
            <div :class="[$style['loader-5'], $style['center']]"><span></span></div>
          </li>
        </ul>
      </div>

      <button v-if="!isLoading1" @click="readMore">Xem thêm</button>
    </div>
    <div :class="$style['news__context-right']">
      <category :class="$style['news__context-right-category']" style="margin-top: 0" />
    </div>
  </div>
</template>

<style module scoped lang="scss">
@import './NewsContext.module.scss';
</style>
