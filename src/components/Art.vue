<template>
  <div id="browzart">
    <div class="container" tabindex="0">
      <img
        id="image"
        :src="resImage.primaryImage"
        :class="[isZoomed ? 'zoomedImg' : '', 'primaryImage', isLoading ? 'hide' : '']"
        :aria-label="[resImage.title ? resImage.title : 'No img title available.', ]"

      />
      <div class="iconbox">
        <div class="buttonContainer">
          <button @click="zoom">
            {{ isZoomed ? '🔽' : '🔼' }}
          </button>
          <button @click="refresh">🔃</button>
          <button @click="addToFavorites">💾</button>
          <button @click="toggleModal"><b>ℹ️</b></button>
          <button @click="fullscreen"><b>🖼️</b></button>
        </div>
      </div>
      <div
        class="modal"
        :class="showModal ? '' : 'hide-modal' "
      >
        <div style="text-align:center; position:relative; top:25%;">
          <p class="title" v-if="resImage.title"><b>{{ resImage.title }}</b></p><br>
          <p v-if="resImage.artistDisplayName">{{ resImage.artistRole +": " + resImage.artistDisplayName}}</p>
          <p v-else>Artist: unknown</p>
          <p v-if="resImage.artistDidsplayBio">{{ resImage.artistDidsplayBio}}</p><br>
          <p v-if="resImage.period">{{ resImage.period}}</p><br>
          <p v-if="resImage.department" style="font-size: 10px;"><i>{{ "Dept: " + resImage.department}}</i></p><br>
          <input type="text" v-model="searchValue" @keyup="getSearchValue"/>
          <button :class="[isDisabled ? 'disable' : '']" @click="refresh" style="position:relative; z-index: 1000;">Search</button><br>
        </div>
        <div class="imagebox">
          <ul style="list-style-type: none;">
            <li v-for="(favorites, index) in favoriteArray" :key="index">
              <a :href="favoriteArray[index]" target="_blank" >
                <img :src="favoriteArray[index]" height="80vw" class="thumbPreview" style="margin: 5px;" />
              </a>
            </li>
          </ul>
        </div>
        <div class="iconbox">
          <div class="buttonContainer">
            <button @click="toggleModal">X</button>
          </div>
        </div>
      </div>
    </div>
    <div v-if="isLoading">
      <BaseSpinner />
      <p style="position:absolute; top: 53vh; left: 48vw; padding-left: 0.5rem; font-size:0.75rem; color: white;"> Loading </p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import BaseSpinner from './BaseSpinner.vue'

export default {
  components: {
    BaseSpinner
  },
  data () {
    return {
      favX: 0,
      favoriteCount: 0,
      favoriteArray: [],
      isLoading: false,
      isZoomed: false,
      isMetLogo: false,
      imageArray: [],
      isDisabled: false,
      resImage: '',
      searchValue: "paintings",
      showModal: false,
      showCursor: true,
      timerOn: false,
      timeAmount: (60000 * 60),
      isFullscreen: false,
    }
  },
  created () {
    return this.process();
  },
  mounted () {
    window.addEventListener('keydown', (e) => {
      if (e.key == 'Escape') {
       this.cursorShow();
      }
    });
  },
  methods: {
    cursorShow () {
      if (!this.fullScreen) {
        this.fullScreen = true;
      }
    },
    async getPrimaryImage (id) {
      if( this.randomId > 0){
        const response = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/objects/' + id)
        if (response.data.primaryImage === '') {
          this.process();
        }else {
          this.resImage = response.data
          this.imageKeep(this.resImage.primaryImage)
          this.isLoading = false;
        }
      }
    },
    async getRandomId () {
      this.isZoomed = false;
      const response = await axios.get('https://collectionapi.metmuseum.org/public/collection/v1/search?q='+ this.searchValue)
      if (response.status ==! 200 ){
        this.searchValue = "paintings";
        this.process ();
      }else {
        return response.data.objectIDs[this.rngId (response.data.objectIDs.length)]
      }
    },
    rngId (max) {
      return Math.floor(Math.random() * max);
    },
    async process () {
      this.isLoading = true;
      this.randomId = await this.getRandomId()
      return this.getPrimaryImage(this.randomId.toString())
    },
    getSearchValue () {
      return this.searchValue;
    },
    zoom () {
      this.isZoomed = !this.isZoomed
    },
    refresh () {
      this.resImage.primaryImage = "";
      this.isZoomed = false;
      this.process();
    },
    imageKeep (image) {
      if (this.imageArray.length < 10) {
        this.imageArray.unshift(image)
      } else {
        this.imageArray.pop();
        this.imageArray.unshift(image)
      }
    },
    addToFavorites () {
      if ( !this.favoriteArray.includes(this.resImage.primaryImage) ){
        this.favoriteCount += 1;
        this.favX += 1
        let favId = 'favorite_' + this.favX
        if ( this.favoriteArray.length === 10) {
          this.favoriteArray.pop();
          this.favoriteCount = 9;
        }
        localStorage.setItem(favId , this.resImage.primaryImage);
        this.favoriteArray.unshift(localStorage.getItem(favId));
      }
    },
    disableButton () {
      this.isDisabled = !this.isDisabled
    },
    toggleModal () {
      this.showModal = !this.showModal
    },
    picReset () {
      this.toggleTimer();
      if(this.timerOn){
        setInterval(function(){
          this.refresh()
        }.bind(this), this.timeAmount);
      }
    },
    toggleTimer () {
      this.timerOn = !this.timerOn
    },
    fullscreen () {
      const elem = document.getElementById("image");

      if (elem.requestFullscreen) {
        elem.requestFullscreen();
      } else if (elem.webkitRequestFullscreen) { /* Safari */
        elem.webkitRequestFullscreen();
      } else if (elem.msRequestFullscreen) { /* IE11 */
        elem.msRequestFullscreen();
      }
      this.isFullscreen = true;
      // this.showCursor = false;
      // this.updateCursor();
    },
    mouseEvent () {
      if (!this.showCursor && this.isFullscreen) {
        this.showCursor = true;
        this.updateCursor();

        setTimeout(() => {
          this.showCursor = false;
        }, 3000);

      }
    },
    updateCursor () {
      const elem = document.getElementById("image");
      if (!this.showCursor) {
        elem.style.cursor = 'none';
      } else {
        elem.style.cursor = 'unset';
      }
    }
  }
}
</script>

//todo
//more api
//cleanup mouse hide
//add logo somewhere
//add option to set timer
//add key for symbols

<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat&family=Open+Sans:wght@400;600&display=swap');

.container {
  background-color: black;
}

.iconbox {
  bottom:20px;
  opacity: 0.3;
  padding: 4px;
  position: fixed;
  width: 160px;
}

.buttonContainer {
  display:flex;
  flex-direction:row;
  justify-content: space-around;
}

.imagebox {
  background-color: rgba(0,0,0);
  right: 0px;
  color:red;
  bottom:20px;
  padding: 4px;
  position: fixed;
  width: 25vw;
  overflow: scroll;
  padding-right: 10vw;
}

.imagebox:hover {
   z-index:100;
   background-image: linear-gradient(to left, rgba(0,0,0,.65), rgba(200,200,200,.15));
    border-radius: 10px;
    -webkit-box-shadow: 5px 5px 11px 1px rgba(0,0,0,0.57);
    box-shadow: 5px 5px 11px 1px rgba(0,0,0,0.57);
}

.imagebox > ul {
  display: flex;
  flex-direction: row;
}

.thumbPreview {
  opacity: 0.5
}

.thumbPreview:hover {
  transform: scale(1.5);
  opacity: 1;
  z-index: 1;
  position: relative;
}

.primaryImage {
  background-color: black;
  border: none;
  height: 100vh;
  object-fit: contain;
  overflow: hidden;
  width: 100vw;
}

.hide {
  background-color: black;
  opacity: 0;
}

.zoomedImg {
  height: 100%;
  width: 100%;
}

.disable {
  pointer-events: none;
}

.hide-modal {
  display: none;
}

.modal {
  color: white;
  background: black;
  opacity: .75;
  height: 90vh;
  width: 90vw;
  position: fixed;
  z-index: 10000;
  top:0;
  margin: 0px auto;
  transform:translate(5vw, 5vh)
}

</style>
