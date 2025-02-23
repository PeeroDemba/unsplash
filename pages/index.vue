<template>
  <main>
    <section class="top">
      <form
        @submit="handleSearch"
        v-show="loading === false && status !== 'idle'"
      >
        <div class="icon-container">
          <font-awesome :icon="faSearch" class="icon" />
        </div>
        <input type="text" placeholder="Search for photo" v-model="search" />
      </form>
      <div class="search">
        <p v-if="loading === true">
          Searching for <span>"{{ search }}"</span>
        </p>
        <p v-else-if="loading === false && status === 'idle'">
          Search results for <span>"{{ search }}"</span>
        </p>
      </div>
    </section>
    <section class="body">
      <div v-if="status === 'pending' || loading === true">
        <div
          v-for="(_, index) in 30"
          :key="index"
          class="items"
          :class="{
            first: index === 0 || index === 9 || index === 18 || index === 27,
            second: index === 1 || index === 10 || index === 19 || index === 28,
            third: index === 2 || index === 11 || index === 20 || index === 29,
            fourth: index === 3 || index === 12 || index === 21,
            fifth: index === 4 || index === 13 || index === 22,
            sixth: index === 5 || index === 14 || index === 23,
            seventh: index === 6 || index === 15 || index === 24,
            eighth: index === 7 || index === 16 || index === 25,
            ninth: index === 8 || index === 17 || index === 26,
          }"
        >
          <div class="placeholder-text">
            <p></p>
            <p></p>
          </div>
        </div>
      </div>
      <div v-else>
        <div
          v-for="(element, index) in dataList"
          @click="sliderStart(index)"
          :key="index"
          class="items"
          :class="{
            first: index === 0 || index === 9 || index === 18 || index === 27,
            second: index === 1 || index === 10 || index === 19 || index === 28,
            third: index === 2 || index === 11 || index === 20 || index === 29,
            fourth: index === 3 || index === 12 || index === 21,
            fifth: index === 4 || index === 13 || index === 22,
            sixth: index === 5 || index === 14 || index === 23,
            seventh: index === 6 || index === 15 || index === 24,
            eighth: index === 7 || index === 16 || index === 25,
            ninth: index === 8 || index === 17 || index === 26,
          }"
        >
          <img :src="element.urls.full" alt="" />
          <div>
            <p>{{ element.user.first_name }} {{ element.user.last_name }}</p>
            <p>{{ element.user.location }}</p>
          </div>
        </div>
      </div>
    </section>
    <Transition leave-active-class="animate">
      <section
        class="carousel"
        v-show="shower"
        @click="
          (e) => {
            e.stopPropagation();
            shower = false;
          }
        "
      >
        <Carousel
          ref="myCarousel"
          v-bind="carouselConfig"
          class="carouselcontainer"
          @click="
            (e) => {
              e.stopPropagation();
              shower = true;
            }
          "
        >
          <Slide
            v-for="(slide, index) in dataList"
            :key="index"
            class="carouselitem"
          >
            <img :src="slide.urls.full" alt="" />
            <div>
              <p>{{ slide.user.first_name }} {{ slide.user.last_name }}</p>
              <p>{{ slide.user.location }}</p>
            </div>
          </Slide>

          <template #addons>
            <Navigation>
              <template #prev>
                <span class="arrow"
                  ><font-awesome :icon="faAngleLeft" class="icon"
                /></span>
              </template>
              <template #next>
                <span class="arrow"
                  ><font-awesome :icon="faAngleRight" class="icon"
                /></span>
              </template>
            </Navigation>
          </template>
        </Carousel>
      </section>
    </Transition>
  </main>
</template>

<script lang="ts" setup>
import {
  faSearch,
  faAngleLeft,
  faAngleRight,
} from "@fortawesome/free-solid-svg-icons";
import "vue3-carousel/carousel.css";
import { Carousel, Slide, Navigation } from "vue3-carousel";

const carouselConfig = {
  itemsToShow: 1,
  breakpointMode: "carousel" as "carousel",
  preventExcessiveDragging: true,
  mouseDrag: false,
  touchDrag: true,
};

let search = ref("");
let dataList = ref();
let loading = ref(false);
let shower = ref(false);
let myCarousel = ref<HTMLDivElement>();

async function handleSearch(e: Event) {
  e.preventDefault();
  if (search.value === "") {
    refresh();
  } else {
    status.value = "idle";
    loading.value = true;
    dataList.value = [];
    const response = await fetch(
      `https://api.unsplash.com/search/photos?page=1&per_page=30&query=${search.value}`,
      {
        headers: {
          "Accept-Version": "v1",
          Authorization: `Client-ID ${import.meta.env.VITE_ACCESS_KEY}`,
        },
      }
    );
    const data = response.json();
    data.then((value) => {
      dataList.value = value.results;
    });
    setTimeout(() => {
      loading.value = false;
    }, 5000);
  }
}

const { status, refresh } = useAsyncData("photos", async () => {
  const response = await fetch(
    "https://api.unsplash.com/photos?page=1&per_page=30",
    {
      headers: {
        "Accept-Version": "v1",
        Authorization: `Client-ID ${import.meta.env.VITE_ACCESS_KEY}`,
      },
    }
  );
  const data = response.json();
  data.then((value) => {
    dataList.value = value;
  });
});

console.log(myCarousel);

async function sliderStart(id: number) {
  shower.value = true;
  await nextTick();
  //  @ts-expect-error
  myCarousel.value?.slideTo(id, false);
}
</script>

<style lang="scss">
html,
body,
p {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: Arial, Helvetica, sans-serif;
}

@keyframes carousel {
  0% {
    scale: 1;
    opacity: 1;
  }

  100% {
    scale: 0;
    opacity: 0;
  }
}

main {
  .animate {
    animation: carousel 0.75s ease-in-out 0.1s;
  }

  & > section.carousel {
    background-color: rgba(0, 0, 0, 0.336);
    width: 100%;
    height: 100vh;
    position: fixed;
    top: 0;
    left: 0;
    display: flex;
    justify-content: center;
    align-items: center;

    & .arrow {
      background-color: white;
      border-radius: 100%;
      padding: 12px;
      padding-left: 18px;
      padding-right: 18px;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    & .carouselcontainer {
      width: 95%;

      & .carouselitem {
        height: 80vh;
        width: 90vw;
        display: flex;
        flex-direction: column;

        & img {
          width: 90vw;
          object-fit: cover;
          height: 80%;
          border-top-left-radius: 1rem;
          border-top-right-radius: 1rem;
        }

        & div {
          height: 20%;
          width: 90vw;
          background-color: white;
          border-bottom-left-radius: 1rem;
          border-bottom-right-radius: 1rem;
          padding: 1.5rem;
          display: flex;
          flex-direction: column;
          justify-content: center;
          gap: 4px;

          & p:first-child {
            font-weight: 500;
            font-size: 22px;
          }
        }
      }
    }
  }

  & > section.top {
    background-color: #ddd;
    height: 20rem;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;

    & form {
      position: relative;
    }

    & .search {
      font-size: 2.5rem;
      font-weight: 500;
      width: 70vw;

      & p {
        color: rgb(9, 41, 82);
      }

      & span {
        color: #999;
      }
    }

    & .icon-container {
      position: absolute;
      top: 24px;
      left: 36px;
    }

    & .icon {
      color: #bbb;
      height: 1.5rem;
    }

    & input {
      height: 4rem;
      border-radius: 1rem;
      padding-top: 4px;
      padding-bottom: 4px;
      padding-right: 4px;
      padding-left: 96px;
      border: none;
      box-shadow: 1px 1px 0.5px 1px #bbb;
      width: 70vw;
      font-size: 1.3rem;
    }
  }

  & > section.body {
    padding-left: 5vw;
    padding-right: 5vw;
    margin-bottom: 5rem;
    margin-top: -3.5rem;

    & > div {
      width: 100%;
      display: flex;
      flex-direction: column;
      gap: 48px;

      & .items {
        position: relative;
        border-radius: 0.5rem;
        height: 30rem;

        & img {
          width: 100%;
          border-radius: 1rem;
          height: 100%;
          object-fit: cover;
        }

        & .placeholder-text {
          display: flex;
          flex-direction: column;
          justify-content: end;
          gap: 8px;
          background-color: #eee;

          & p {
            padding: 0.5rem;
            background-color: #ccc;
            border-radius: 0.25rem;

            &:first-child {
              width: 55%;
            }

            &:last-child {
              width: 35%;
            }
          }
        }

        & > div {
          color: white;
          height: calc(100% - 3rem);
          width: calc(100% - 3rem);
          backdrop-filter: brightness(0.85);
          border-radius: 0.5rem;
          position: absolute;
          top: 0;
          left: 0;
          padding: 1.5rem;
          display: flex;
          flex-direction: column;
          justify-content: end;
          gap: 4px;

          & p:first-child {
            font-weight: 500;
            font-size: 22px;
          }
        }
      }
    }
  }
}

@media (min-width: 768px) {
  main {
    & > section.carousel {
      background-color: rgba(0, 0, 0, 0.336);
      width: 100%;
      height: 100vh;
      position: fixed;
      top: 0;
      left: 0;
      display: flex;
      justify-content: center;
      align-items: center;

      & .arrow {
        background-color: white;
        border-radius: 100%;
        padding: 12px;
        padding-left: 18px;
        padding-right: 18px;
        display: flex;
        justify-content: center;
        align-items: center;
      }

      & .carouselcontainer {
        width: 80%;

        & .carouselitem {
          height: 80vh;
          width: 35rem;
          display: flex;
          flex-direction: column;

          & img {
            width: 35rem;
            object-fit: cover;
            height: 80%;
            border-top-left-radius: 1rem;
            border-top-right-radius: 1rem;
          }

          & div {
            height: 20%;
            width: 35rem;
            background-color: white;
            border-bottom-left-radius: 1rem;
            border-bottom-right-radius: 1rem;
            padding: 1.5rem;
            display: flex;
            flex-direction: column;
            justify-content: center;
            gap: 4px;

            & p:first-child {
              font-weight: 500;
              font-size: 22px;
            }
          }
        }
      }
    }

    & > section.body {
      padding-left: 5vw;
      padding-right: 5vw;
      margin-bottom: 5rem;
      margin-top: -3.5rem;

      & > div {
        width: 100%;
        gap: 48px;
        flex-wrap: wrap;
        flex-direction: row;
        justify-content: center;

        & .items {
          position: relative;
          border-radius: 0.5rem;
          width: 20rem;
        }
      }
    }
  }
}

@media (min-width: 1024px) {
  main {
    & > section.body {
      padding-left: 5vw;
      padding-right: 5vw;
      margin-bottom: 5rem;
      margin-top: -3.5rem;

      & > div {
        width: 100%;
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        grid-auto-rows: 3rem;
        gap: 48px;

        & .items {
          position: relative;
          border-radius: 0.5rem;
          height: 100%;
          width: 100%;

          &.first {
            grid-row-start: span 4;
            grid-column: 1 / span 1;
          }

          &.second {
            grid-row-start: span 6;
            grid-column: 2 / span 1;
          }

          &.third {
            grid-row-start: span 5;
            grid-column: 3 / span 1;
          }

          &.fourth {
            grid-row-start: span 5;
            grid-column: 1 / span 1;
          }

          &.fifth {
            grid-row-start: span 4;
            grid-column: 2 / span 1;
          }

          &.sixth {
            grid-row-start: span 6;
            grid-column: 3 / span 1;
          }

          &.seventh {
            grid-row-start: span 6;
            grid-column: 1 / span 1;
          }

          &.eighth {
            grid-row-start: span 5;
            grid-column: 2 / span 1;
          }

          &.ninth {
            grid-row-start: span 4;
            grid-column: 3 / span 1;
          }
        }
      }
    }
  }
}

@media (min-width: 1280px) {
  main {
    & > section.body {
      padding-left: 112px;
      padding-right: 112px;
      margin-bottom: 5rem;
      margin-top: -3.5rem;
    }
  }
}

@media (min-width: 1440px) {
  main {
    & > section.body {
      padding-left: 224px;
      padding-right: 224px;
      margin-bottom: 5rem;
      margin-top: -3.5rem;
    }
  }
}
</style>
