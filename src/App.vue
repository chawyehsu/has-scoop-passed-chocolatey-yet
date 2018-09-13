<template>
  <div id="app">
    <github-corner/>
    <p>Has Scoop passed Chocolatey yet?</p>
    <template v-if="error">
      <h1 class="error">Error</h1>
      <p>
        Couldn't retrieve any data.
        The API rate limits might have kicked in. Just wait a bit and try again.
      </p>
    </template>
    <template v-else-if="loaded">
      <h1 v-if="!tie">{{ scoopHasPassedChoco ? 'YES' : 'NO' }}</h1>
      <h1 :class="{ pad : tie }" v-else>TIE!</h1>
      <p>
        <small v-if="!scoopHasPassedChoco && !tie" class="away">
          Only {{ chocoStars - scoopStars | formatNumber }} {{ chocoStars - scoopStars === 1 ? 'star' : 'stars'}} away!
        </small>
        <small v-else-if="scoopHasPassedChoco && !tie" class="ahead">
          Ahead by {{ scoopStars - chocoStars | formatNumber }} {{ scoopStars - chocoStars === 1 ? 'star' : 'stars'}}!
        </small>
      </p>
      <ul>
        <li>
          <a :href="repos.scoop.url" target="_blank" title="Go to Scoop's GitHub repository">
            <scoop-icon/>
            <span>{{ scoopStars | formatNumber }}</span>
            <star-icon/>
          </a>
        </li>
        <li>
          <a :href="repos.choco.url" target="_blank" title="Go to Chocolatey's GitHub repository">
            <choco-icon/>
            <span>{{ chocoStars | formatNumber }}</span>
            <star-icon/>
          </a>
        </li>
      </ul>
      <span class="reload" @click="reload">
        <svg :class="{ reloading }" xmlns="http://www.w3.org/2000/svg" width="24" height="24"><path fill="#333333" d="M19 8l-4 4h3c0 3.31-2.69 6-6 6a5.87 5.87 0 0 1-2.8-.7l-1.46 1.46A7.93 7.93 0 0 0 12 20c4.42 0 8-3.58 8-8h3l-4-4zM6 12c0-3.31 2.69-6 6-6 1.01 0 1.97.25 2.8.7l1.46-1.46A7.93 7.93 0 0 0 12 4c-4.42 0-8 3.58-8 8H1l4 4 4-4H6z"/><path d="M0 0h24v24H0z" fill="none"/></svg>
      </span>
    </template>
    <p v-else>Loading...</p>
  </div>
</template>

<script>
import axios from 'axios'
import GithubCorner from './components/GithubCorner'
import { ScoopIcon, ChocoIcon, StarIcon } from './components/icons'

export default {
  name: 'App',

  data() {
    return {
      repos: {
        scoop: {
          endpoint: 'https://api.github.com/repos/lukesampson/scoop',
          url: 'https://github.com/lukesampson/scoop',
          stars: 0
        },
        choco: {
          endpoint: 'https://api.github.com/repos/chocolatey/choco',
          url: 'https://github.com/chocolatey/choco',
          stars: 0
        }
      },
      error: false,
      loaded: false,
      reloading: false
    }
  },

  components: {
    ScoopIcon,
    ChocoIcon,
    StarIcon,
    GithubCorner
  },

  mounted() {
    this.fetchData()
    if ('ontouchstart' in window || navigator.msMaxTouchPoints) {
      document.body.classList.remove('no-touch')
    }
  },

  computed: {
    scoopHasPassedChoco() {
      return this.repos.scoop.stars > this.repos.choco.stars
    },

    scoopStars() {
      return this.repos.scoop.stars
    },

    chocoStars() {
      return this.repos.choco.stars
    },

    tie() {
      return this.repos.scoop.stars === this.repos.choco.stars
    }
  },

  filters: {
    formatNumber(number) {
      return new Intl.NumberFormat().format(number)
    }
  },

  methods: {
    async fetchData() {
      try {
        const { data: resScoop } = await axios.get(this.repos.scoop.endpoint)
        const { data: resChoco } = await axios.get(this.repos.choco.endpoint)

        if (resScoop.stargazers_count && resChoco.stargazers_count) {
          this.repos.scoop.stars = resScoop.stargazers_count
          this.repos.choco.stars = resChoco.stargazers_count
          this.error = false
        } else {
          this.error = true
        }
        this.loaded = true
      } catch (err) {
        // eslint-disable-next-line
        console.log(err)
      }
    },

    async reload() {
      if (this.reloading) return
      this.reloading = true
      await this.fetchData()
      setTimeout(() => {
        this.reloading = false
      }, 900)
    }
  }
}
</script>

<style>

* {
  box-sizing: border-box;
}

::selection {
  background: rgba(0,0,0,0);
}
::-moz-selection {
  background: rgba(0,0,0,0);
}

html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}

body {
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
  text-align: center;
  color: #333;
  background: #efefef;
}

#app {
  width: 300px;
  border: 1px solid #dddddd;
  border-radius: 4px;
  background: #ffffff;
  box-shadow: 0 15px 35px rgba(50,50,93,.1), 0 5px 15px rgba(0,0,0,.07);
  overflow: hidden;
  position: relative;
}

h1 {
  font-size: 100px;
  margin: 10px 0;
}

ul {
  margin: 0;
  padding: 0;
  display: flex;
}

li {
  list-style-type: none;
  width: 50%;
}

li a {
  display: flex;
  align-items: center;
  justify-content: center;
  border-top: 1px solid #dddddd;
  padding: 10px;
  text-decoration: none;
  color: #333;
}

.no-touch li:hover {
  background: #eeeeee;
}

li a > svg {
  display: block;
  width: 22px;
  height: 22px;
}

li a > * {
  margin: 5px;
}

li:last-of-type {
  border-left: 1px solid #dddddd;
}

h1.error {
  font-size: 2em;
}

h1.pad {
  margin-bottom: 30px;
}

p {
  padding: 0 1em;
}

.away, .ahead {
  display: block;
  margin-bottom: 40px;
}

.reload {
  position: absolute;
  left: 50%;
  margin-left: -20px;
  bottom: 30px;
  background: #ffffff;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid #dddddd;
  border-radius: 50%;
}

.reload svg {
  transform-origin: center center;
}

.no-touch .reload:hover {
  cursor: pointer;
  background: #eeeeee;
}

.reloading {
  animation: rotate 1s infinite ease-in-out;
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(-360deg);
  }
}

</style>
