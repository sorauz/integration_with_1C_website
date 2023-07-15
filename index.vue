<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <link href="https://fonts.googleapis.com/css?family=Roboto:100,300,400,500,700,900" rel="stylesheet">
  <link href="https://cdn.jsdelivr.net/npm/@mdi/font@4.x/css/materialdesignicons.min.css" rel="stylesheet">
  <link href="https://cdn.jsdelivr.net/npm/vuetify@2.x/dist/vuetify.min.css" rel="stylesheet">
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, minimal-ui">
</head>
<body>
<div id="app">
  <v-app>
    <v-main>
      <v-container>
        <v-card
            :loading="loading"
            class="mx-auto my-12"
            max-width="374"
            v-for="(product, i) in products"
            :key="i"
        >
          <v-img
              height="250"
              :src="product.img"
          ></v-img>

          <v-card-title>{{ product.title }}</v-card-title>

          <v-card-subtitle>
            Цена: {{ product.price }} сум.
          </v-card-subtitle>

          <v-divider class="mx-4"></v-divider>

          <v-card-text>
            {{ product.desk }}
          </v-card-text>

          <template slot="progress">
            <v-progress-linear
                color="deep-purple"
                height="10"
                indeterminate
            ></v-progress-linear>
          </template>
        </v-card>

        <div class="observer" ref="observer"></div>
      </v-container>
    </v-main>
  </v-app>
</div>

<script src="https://cdn.jsdelivr.net/npm/vue@2.x/dist/vue.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vuetify@2.x/dist/vuetify.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script>

  new Vue({
    el: '#app',
    vuetify: new Vuetify(),
    data() {
      return {
        loading: false,
        lastCode: 0,
        products: [],
        observer: null
      }
    },
    methods: {
      async load() {
        this.loading = true
        try {
          const url = this.lastCode === 0 ? 'http://localhost/my_servece/hs/vue/products' : `http://localhost/my_servece/hs/vue/products?lastcode=${this.lastCode}`
          const response = await axios.get(url, {
            credential: true,
            auth: {
              username: 'root',
              password: 1
            }
          })

          this.lastCode = response.data.lastcode
          this.products = this.products.concat(response.data.products)
          console.log(response.data)
        } catch (e) {
          console.log(e)
        } finally {
          this.loading = false
        }
      }
    },
    mounted () {
      if (window['IntersectionObserver']) {
        this.observer = new IntersectionObserver(([entry]) => {
          if (entry && entry.isIntersecting) {
            this.load()
          }
        })

        this.observer.observe(this.$refs.observer)
      } else {
        this.loading = true
      }
    }
  })
</script>
</body>
</html>