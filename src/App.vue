<template>
  <div id="app">
    <div
      class="bg-gradient-to-r from-purple-400 via-pink-500 to-red-500 h-screen w-full flex flex-col sm:flex-row items-center justify-center p-10 space-y-5 sm:space-x-5 "
    >
      <div
        class="border h-1/2 bg-white shadow-my w-96 text-center rounded-xl"
        style="height: 600px"
      >
        <div class="overflow-hidden flex flex-col justify-around items-center">
          <div
            class="w-72 h-52 border rounded-xl mt-5 bg-no-repeat bg-cover bg-center"
            :style="{ backgroundImage: `url(${currentSong.img})` }"
          ></div>
          <div class="mt-3 text-2xl font-mono font-bold items-center w-full">
            {{ currentSong.name }}
          </div>
          <div class="my-3 text-lg">{{ currentSong.by }}</div>
        </div>

        <div
          v-if="buttonsAtFirst"
          class="flex items-center justify-center border h-1/4"
        >
          <button
            v-if="songs.indexOf(currentSong) == 0 ? false : true"
            v-on:click="previousSong(currentSong)"
            class="bg-white shadow-my p-4 m-4 rounded-full hover:bg-gray-100 cursor-pointer focus:outline-none"
          >
            <svg
              class="fill-current text-green-600"
              width="30"
              height="30"
              viewBox="0 0 24 24"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path d="M2 7H5V17H2V7Z" fill="currentColor" />
              <path d="M6 12L13.0023 7.00003V17L6 12Z" fill="currentColor" />
              <path
                d="M21.0023 7.00003L14 12L21.0023 17V7.00003Z"
                fill="currentColor"
              />
            </svg>
          </button>

          <button
            v-if="!isPlaying"
            v-on:click="playSong(currentSong, $event)"
            class="bg-white p-4 m-4 rounded-full shadow-my hover:bg-gray-100 cursor-pointer focus:outline-none"
          >
            <svg
              class="fill-current text-green-600"
              width="35"
              height="35"
              viewBox="0 0 24 24"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M15 12.3301L9 16.6603L9 8L15 12.3301Z"
                fill="currentColor"
              />
            </svg>
          </button>

          <button
            v-else
            v-on:click="pauseSong(currentSong)"
            class="bg-white p-4 m-4 rounded-full shadow-my hover:bg-gray-100 cursor-pointer focus:outline-none"
          >
            <svg
              class="fill-current text-green-600"
              width="35"
              height="35"
              viewBox="0 0 24 24"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path d="M11 7H8V17H11V7Z" fill="currentColor" />
              <path d="M13 17H16V7H13V17Z" fill="currentColor" />
            </svg>
          </button>

          <button
            v-if="currentSong == songs[songs.length - 1] ? false : true"
            v-on:click="nextSong(currentSong)"
            class="bg-white p-4 m-4 rounded-full shadow-my hover:bg-gray-100 cursor-pointer focus:outline-none"
          >
            <svg
              class="fill-current text-green-600"
              width="30"
              height="30"
              viewBox="0 0 24 24"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path d="M21.0023 17H18.0023V7H21.0023V17Z" fill="currentColor" />
              <path d="M17.0023 12L10 17V7L17.0023 12Z" fill="currentColor" />
              <path d="M2 17L9.00232 12L2 7V17Z" fill="currentColor" />
            </svg>
          </button>
        </div>

        <div class="h-1/4 flex flex-col justify-center items-center ">
             
          <div class="flex justify-between items-center w-72 font-bold -mt-5">
              <div>{{songTime | fixNumber1}}</div>
              <div v-if="Number.isNaN(player.duration)==true?false:true">{{player.duration | fixNumber2}}</div>
              <div v-else>0.00</div>
          </div>
          <div
            class="h-2 border border-black m-2 rounded-full"
            style="width: 300px"
            v-on:mousedown.left="changeDuration($event)"
          >
            <div
              class="h-full bg-green-600 rounded-full"
              :style="{ width: progressBar + '%' }"
            ></div>
          </div>

          <input
            class="mb-5"
            type="range"
            name="volume"
            id="volume"
            min="0"
            max="100 "
            v-model="songVolume"
            v-on:change="changeVolume(songVolume)"
          />
   
        </div>
      </div>

      <div
        class="shadow-my bg-white text-center w-96 overflow-auto rounded-xl h-96"
      >
        <div v-for="(song, index) in songs" :key="index">
          <div
            class="p-8 flex items-center text-xl font-mono hover:bg-green-500 hover:text-gray-100 cursor-pointer overflow-hidden"
            v-on:click="current(song)"
          >
            <div class="h-full w-1/4 flex justify-items-center items-center">
              <div
                class="w-16 h-16 rounded-xl bg-no-repeat bg-cover bg-center"
                :style="{ backgroundImage: `url(${song.img})` }"
              ></div>
            </div>
            <div class="h-full w-3/4">
              <p>
                <span class="font-bold text-2xl">{{ song.name }}</span> -
                <span class=" ">{{ song.by }}</span>
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",

  data() {
    return {
      isPlaying: false,
      songVolume: 100,
      progressBar: 0,
      songTime: 0,
     

      buttonsAtFirst: false,

      currentSong: {},

      songs: [
        {
          name: "Piratos",
          by: "by Wowa & Chris Rede",
          src: require("./assets/musics/Piratos _ www.wowa.me.mp3"),
          img:
            "https://images.pexels.com/photos/206359/pexels-photo-206359.jpeg?auto=compress&cs=tinysrgb&dpr=3&h=750&w=1260",
        },
        {
          name: "Ambisax",
          by: "by RRound",
          src: require("./assets/musics/Ambisax - unminus.com.mp3"),
          img:
            "https://images.pexels.com/photos/417074/pexels-photo-417074.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=750&w=1260",
        },
        {
          name: "All Alone",
          by: "by Hospitalized",
          src: require("./assets/musics/Hospitalized - All Alone _ Dubstep _ 138 BPM _ wowa.me.mp3"),
          img:
            "https://images.pexels.com/photos/1910225/pexels-photo-1910225.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=750&w=1260",
        },
        {
          name: "Kosu",
          by: "by Shock of Daylight",
          src: require("./assets/musics/Kosu - Unminus.com.mp3"),
          img:
            "https://images.pexels.com/photos/54539/pexels-photo-54539.jpeg?auto=compress&cs=tinysrgb&dpr=1&w=500",
        },
        {
          name: "Palmtrees",
          by: "by JNGS ",
          src: require("./assets/musics/Palmtrees _ www.wowa.me.mp3"),
          img:
            "https://images.pexels.com/photos/219998/pexels-photo-219998.jpeg?auto=compress&cs=tinysrgb&dpr=1&w=500",
        },
      ],

      player: new Audio(),
    };
  },

  methods: {
    current(song) {
      this.currentSong = song;
      this.player.src = song.src;
      this.buttonsAtFirst = true;
      this.isPlaying = false;
      

      console.log(song);
    },

    changeVolume(songVolume) {
      this.player.volume = songVolume / 100;
    },

    changeDuration(event) {
      console.log(event.offsetX);

      this.player.currentTime = (event.offsetX * this.player.duration) / 300;
    },

    playSong() {
      this.player.ontimeupdate = () => {
        this.songTime = this.player.currentTime;
        this.progressBar = (this.songTime * 100) / this.player.duration;

        if (this.player.ended) {
          this.isPlaying = false;
          this.player.currentTime = 0;
        }
      };

      this.player.play().catch(() => void 0);
      this.isPlaying = true;
      this.player.volume = this.songVolume / 100;

      console.log(this.player.currentTime);

      console.log(this.player.duration);
    },

    pauseSong(music) {
      this.player.pause();
      this.isPlaying = false;
      console.log(music.name);
      console.log(this.player.currentTime);
    },

    nextSong(song) {
      let nextSongIndex = this.songs.indexOf(song) + 1;
      let nextSong = this.songs[nextSongIndex];
      this.currentSong = nextSong;
      this.player.src = this.currentSong.src;
      this.isPlaying = false;
      console.log(nextSongIndex);
    },

    previousSong(song) {
      let previousSongIndex = this.songs.indexOf(song) - 1;
      let previousSong = this.songs[previousSongIndex];
      this.currentSong = previousSong;
      this.player.src = this.currentSong.src;
      this.isPlaying = false;
      console.log(previousSongIndex);
    },
  },

  filters:{
    fixNumber1(number1){

      return (number1/60).toFixed(2)

    },
    fixNumber2(number2){

       return (number2/60).toFixed(2)

    }

  },

  mounted() {
    this.current(this.songs[0]);
  },
};
</script >



<style src="./assets/tailwind.css" scoped>
</style>



