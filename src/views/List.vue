<template>
  <div>
    <gallery :images="images" :index="index" @close="index = null"></gallery>

    <div class="wrapper">
      <p v-if="loading" class="text-centered">
        Loading...
      </p>
      <ul v-else class="image-card-grid">
        <image-card
          v-for="image in images"
          :key="image.id"
          :image="image"
          @click.native="() => showImg(images.indexOf(image))"
        />
      </ul>
      <infinite-loading @infinite="infiniteHandler"></infinite-loading>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import VueGallery from 'vue-gallery';
import InfiniteLoading from 'vue-infinite-loading';
import config from '../../config';
import ImageCard from '@/components/ImageCard.vue';

export default {
  name: 'home',
  components: {
    ImageCard,
    InfiniteLoading,
    gallery: VueGallery,
  },
  data() {
    return {
      loading: false,
      tag: '',
      visible: false,
      index: null,
      images: [],
      page: 1,
    };
  },
  // created() {
  //   this.loading = true;
  //   this.fetchImages()
  //     .then((response) => {
  //       // eslint-disable-next-line array-callback-return
  //       response.data.photos.photo.map((image) => { this.images.push(image.url_n); });
  //       this.loading = false;
  //     })
  //     .catch((error) => {
  //       console.log('An error ocurred: ', error);
  //     });
  // },
  methods: {
    fetchImages() {
      return axios({
        method: 'get',
        url: 'https://api.flickr.com/services/rest',
        params: {
          method: 'flickr.galleries.getPhotos',
          api_key: config.api_key,
          gallery_id: '66911286-72157647277042064',
          tags: this.tag,
          extras: 'url_n, owner_name, date_taken, views',
          page: this.page,
          format: 'json',
          nojsoncallback: 1,
          per_page: 12,
        },
      });
    },
    showImg(index) {
      this.index = parseInt(index, 10);
      this.visible = true;
    },
    infiniteHandler($state) {
      this.fetchImages()
        .then(({ data }) => {
          if (data.photos.photo.length) {
            this.page += 1;
            // eslint-disable-next-line array-callback-return
            data.photos.photo.map((image) => { this.images.push(image.url_n); });
            $state.loaded();
          } else {
            $state.complete();
          }
        });
    },
  },
};
</script>

<style lang="scss">
  .screen-reader-only {
    height: 1px;
    width: 1px;
    position: absolute;
    left: -100000px;
  }
  .text-centered {
    text-align: center;
  }
  .wrapper {
    margin: 0 auto;
    max-width: 800px;
    @media only screen and (max-width: 799px) {
      max-width: 100%;
      margin: 0 1.5rem;
    }
  }
  .image-card-grid {
    list-style: none;
    margin: .5rem 0;
    padding: 0;
    display: flex;
    align-items: flex-start;
    flex-wrap: wrap;
  }
  .navbar {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 1rem;
    background: #F0F0F0;
  }
  .searchbar {
    width: 300px;
    display: flex;
    align-items: center;
    justify-content: center;
    @media only screen and (max-width: 549px) {
      width: 100%;
      label {
        width: 80%;
      }
    }
  }
  .searchbar-input {
    padding: .5rem 1rem;
    border-radius: 20px;
    font-size: 1rem;
    border: 1px solid gray;
    min-width: 300px;
    @media only screen and (max-width: 549px) {
      min-width: 0;
      width: 100%;
    }
  }
  .btn {
    padding: .5rem 1rem;
    font-size: 1rem;
    border-radius: 20px;
    background: transparent;
    border: none;
  }
  .btn--green {
    background: #42b983;
    color: white;
    font-weight: bold;
  }
  .btn--go {
    padding: .5rem 2rem;
    margin-left: 1rem;
  }
</style>
