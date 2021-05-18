<template>
  <div>
    <form class="searchForm" @submit="submitForm">
      <h1>{{ msg }} <span @click="clearCache" class="clearCache">😉 </span></h1>
        <div class="row webAddress">
          <input type=text v-model="addressField" placeholder="Web address" />
        </div>
        <div class="row searchkey">
          <input type=text v-model="searchField" placeholder="Search" />
        </div>
        <div class="row button">
          <button type="submit" :disabled="isDisabled">Let's get it</button>
        </div>
        <div class="row rank"  v-if="searchRanks.length">
          <cite class="cite">This result <strong>{{cached ? 'is' : 'will now be'}} cached</strong> due to the Google request limiter, You can reload the 'server' to get a fresh result or click on the wink emoji before peforming a new search</cite>
          <h3> You're ranking top @</h3>
          <span class="leadRank">{{searchRanks[0].index+1}}</span>
          <div class="otherRanks" v-if="searchRanks.slice(1).length">Also ranking @ 
            <span v-for="rank in searchRanks.slice(1)" :key="rank.index">{{rank.index+1}}</span>
          </div>
        </div>
    </form>
    <div id="toast" ref="toast">
      <div id="img">🥷</div>
      <div id="desc">{{toastMessage}}</div>
    </div>
  </div>
</template>

<script>
/*eslint no-unused-vars: "error"*/
import Vue from 'vue';
import axios from 'axios'
import VueAxios from 'vue-axios'
Vue.use(VueAxios, axios);

export default {
  name: "SearchField",
  props: {
    msg: String,
  },

  data: () => {
    return {
      searchField: '',
      addressField: '',
      searchRanks: [],
      cached: false,
      serverAddress: 'http://localhost:3000',
      toastMessage: 'A notification message..',
    }
  },

  computed: {
    isDisabled() {
      return this.searchField === '' || 
        this.addressField === ''
    }
  },

  methods: {
    notify(message) {
      const toast = this.$refs.toast;
      this.toastMessage = message;
      toast.className = "show";
      setTimeout(() => { 
        toast.className = toast.className.replace('show', ''); 
      }, 5000);
    },

    clearCache() {
      Vue.axios
        .get(`${this.serverAddress}/clear`)
        .then(() => {
          this.searchRank = 0;
          this.notify('Server cache now cleared..');
        });
    },

    scrollTo(x = 0, y = 0) {
      history.scrollRestoration = 'manual';
      window.scrollTo({
        top: y, left: x, behaviour: 'smooth'
      });
    },

    submitForm(e) {
      e.preventDefault();
      Vue.axios
        .get(`${this.serverAddress}/search`, { 
          params: { q: this.searchField } 
        })
        .then(response => {
          console.log("response arrived successfully");
          this.cached = response.data.cached;
          this.getRanking(response.data);

          if(this.cached) this.notify('This result is cached..');
          else this.notify('Easy peasy lemon squeezy..');
        });
    },

    getRanking(data) {
      this.searchRanks = data.results.map((item, index) => {
        return { index: index, url: item.link }
      }).filter(item => item.url.includes(this.addressField));

      if(this.searchRanks.length) {
        this.scrollTo(0, document.body.scrollHeight);
        console.log(`Ranking @ ${this.searchRanks.map(r => r.index+1)}`);
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 30px 0 0;
}

.row {
  padding: 10px;
}

.rank {
  margin-bottom: 30px;
}

.rank > .cite {
  width: 550px;
  max-width: 80%;
  display: block;
  font-size: 0.9em;
  margin: 30px auto 0;
}

.rank .leadRank {
  display: inline-block;
  font-size: 10em;
  line-height: 1.3em;
  padding: 0 50px;
  margin-top: 10px;
  border: 1px solid #d6d6d6;
}

.otherRanks {
  padding: 10px;
  display: block;
  font-size: 0.85em;
}

.otherRanks > span {
  border: 1px solid #c5c5c5;
  border-radius: 3px;
  padding: 0 5px;
}

.clearCache {
  display: inline-block;
  cursor: pointer;
}

.clearCache:hover {
  animation: shake 0.5s;
  transition-property: all;
  transition-property: transform;
  animation-iteration-count: infinite;
}

input[type=text] {
  padding: 15px;
  width: 500px;
  max-width: 80%;
  color: #1f1f1f;
  font-size: 1em;
  border: 1px solid #00455a;
  font-family: Courier New,Courier,Lucida Sans Typewriter,Lucida Typewriter,monospace;
}

button {
  padding: 10px 30px;
  color: white;
  font-weight: 700;
  font-size: 1em;
  border-radius: 3px;
  background-color: #5e98aa;
  transition: all 300ms;
  border: 3px solid transparent;
}

button:hover:enabled {
  border: 3px solid rgb(20, 87, 126);
  cursor: pointer;
  background-color: #193138;
}

button:disabled  {
  background-color: #646464;
}

@keyframes shake {
  0% { transform: translate(1px, 1px) rotate(0deg); }
  10% { transform: translate(-1px, -2px) rotate(-1deg); }
  20% { transform: translate(-2px, 0px) rotate(1deg); }
  30% { transform: translate(2px, 2px) rotate(0deg); }
  40% { transform: translate(1px, -1px) rotate(1deg); }
  50% { transform: translate(-1px, 2px) rotate(-1deg); }
  60% { transform: translate(-2px, 1px) rotate(0deg); }
  70% { transform: translate(2px, 1px) rotate(-1deg); }
  80% { transform: translate(-1px, -1px) rotate(1deg); }
  90% { transform: translate(1px, 2px) rotate(0deg); }
  100% { transform: translate(1px, -2px) rotate(-1deg); }
}


#toast {
  visibility: hidden;
  max-width: 50px;
  height: 50px;
  /*margin-left: -125px;*/
  margin: auto;
  background-color: #333;
  color: #fff;
  text-align: center;
  border-radius: 2px;

  position: fixed;
  z-index: 1;
  left: 0;right:0;
  bottom: 30px;
  font-size: 17px;
  white-space: nowrap;
}

#toast #img{
	width: 50px;
	height: 50px;
  float: left;
  padding-top: 16px;
  padding-bottom: 16px;
  box-sizing: border-box;
  background-color: #111;
  color: #fff;
}

#toast #desc{
  color: #fff;
  padding: 16px;
  overflow: hidden;
	white-space: nowrap;
}

#toast.show {
  visibility: visible;
  -webkit-animation: fadein 0.5s, expand 0.5s 0.5s,stay 3s 1s, shrink 0.5s 2s, fadeout 0.5s 2.5s;
  animation: fadein 0.5s, expand 0.5s 0.5s,stay 3s 1s, shrink 0.5s 4s, fadeout 0.5s 4.5s;
}

@-webkit-keyframes fadein {
  from {bottom: 0; opacity: 0;} 
  to {bottom: 30px; opacity: 1;}
}

@keyframes fadein {
  from {bottom: 0; opacity: 0;}
  to {bottom: 30px; opacity: 1;}
}

@-webkit-keyframes expand {
  from {min-width: 50px} 
  to {min-width: 350px}
}

@keyframes expand {
  from {min-width: 50px}
  to {min-width: 350px}
}
@-webkit-keyframes stay {
  from {min-width: 350px} 
  to {min-width: 350px}
}

@keyframes stay {
  from {min-width: 350px}
  to {min-width: 350px}
}
@-webkit-keyframes shrink {
  from {min-width: 350px;} 
  to {min-width: 50px;}
}

@keyframes shrink {
  from {min-width: 350px;} 
  to {min-width: 50px;}
}

@-webkit-keyframes fadeout {
  from {bottom: 30px; opacity: 1;} 
  to {bottom: 60px; opacity: 0;}
}

@keyframes fadeout {
  from {bottom: 30px; opacity: 1;}
  to {bottom: 60px; opacity: 0;}
}
</style>
