<template>
  <div class="app">
    <div class="home">
      <div :class="[{ flexStart: step === 1 }, 'wrapper']">
          <transition name="slide">
            <img src="./assets/logo.svg" class="logo" alt="" v-if="step === 1">
          </transition>
          <transition name="fade">
            <HeroImage v-if="step === 0"/>
          </transition>
        <Claim v-if="step === 0" />
        <SearchInput v-model="searchValue" @input="handleInput" :dark="step === 1"/>
          <div class="results" v-if="results && !loading && step === 1">
              <div v-for="item in results">
                  <Item v-for="item in results" :item="item" :key="item.data[0].nasa_id" @click.native = "handleModalOpen(item)"/>
              </div>
          </div>
          <div class="loader" v-if = "step === 1 && loading"></div>
          <Modal v-if="modalOpen" :item="modalItem" @closeModal = "modalOpen = false"/>
      </div>
    </div>
  </div>
</template>

<script>
    import axios from 'axios';
    import debounce from 'lodash.debounce';
    import Claim from '@/components/Claim';
    import SearchInput from '@/components/SearchInput';
    import HeroImage from '@/components/HeroImage';
    import Item from '@/components/Item';
    import Modal from '@/components/Modal';

    const API = 'https://images-api.nasa.gov/search';
    export default {
        name: 'App',
        components: {
            HeroImage,
            Claim,
            SearchInput,
            Item,
            Modal,
        },
        data() {
            return {
                modalOpen: false,
                modalItem: null,
                loading: false,
                step: 0,
                searchValue: '',
                results: [],
            };
        },
        methods: {
            handleModalOpen(item) {
                this.modalOpen = true;
                this.modalItem = item;
            },
            handleInput: debounce(function () {
                this.loading = true;
                console.log(this.searchValue);
                axios.get(`${API}?q=${this.searchValue}&media_type=image`)
                    .then((response) => {
                        this.results = response.data.collection.items;
                        this.loading = false;
                        this.step = 1;
                    })
                    .catch((error) => {
                        console.log(error);
                    });
            }, 500),
        },
    };
</script>

<style lang="scss">
  @import url('https://fonts.googleapis.com/css?family=Montserrat:300,400,600,800&display=swap');

  * {
    box-sizing: border-box;
    -webkit-font-smoothing: antialiased;
    -moz-csx-font-smoothing: grayscale;
  }

  body {
    font-family: "Montserrat", sans-serif;
    margin: 0;
    padding: 0;

  }
  .fade-enter-active, .fade-leave-active {
      transition: opacity .3s ease;
  }

  .fade-enter, .fade-leave-to
  /* .fade-leave-active below version 2.1.8 */ {
      opacity: 0;
  }

  .slide-enter-active, .slide-leave-active {
      transition: margin-top .3s ease;
  }

  .slide-enter, .slide-leave-to
      /* .fade-leave-active below version 2.1.8 */ {
      margin-top: -50px;
  }

  .wrapper {
      margin: 0;
      position: relative;
      padding: 30px;
      width: 100%;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;

      &.flexStart {
          justify-content: flex-start;
      }
    }

    .loader {
        margin-top: 100px;
        display: inline-block;
        width: 64px;
        height: 64px;

        @media (min-width: 768px) {
            width: 90px;
            height: 90px;
        }
    }

    .loader:after {
        content: " ";
        display:block;
        width: 46px;
        height: 46px;
        margin: 1px;
        border-radius: 50%;
        border: 5px solid #1e3d4a;
        border-color: #1e3d4a transparent #1e3d4a transparent;
        animation: loading 1.2s linear infinite;

        @media (min-width: 768px) {
            width: 90px;
            height: 90px;
        }
    }

    @keyframes loading {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }

    .logo{
        position: absolute;
        top: 30px;
    }

    .results {
        margin-top: 50px;
        display: grid;
        grid-template-columns: 1fr 1fr;
        grid-gap: 20px;

        @media (min-width: 768px) {
            width: 90%;
            grid-template-columns: 1fr 1fr 1fr;
        }
    }

</style>
