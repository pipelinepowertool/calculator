<template>
  <v-app>
    <navigation :color="color" :flat="flat"/>
    <v-main class="pt-0">
      <home/>
      <demo/>
      <about/>
      <pricing/>

      <contact/>
    </v-main>
    <v-scale-transition>
      <v-btn
          fab
          v-show="fab"
          v-scroll="onScroll"
          dark
          fixed
          bottom
          right
          color="secondary"
          @click="toTop"
      >
        <v-icon>mdi-arrow-up</v-icon>
      </v-btn>
    </v-scale-transition>
    <ftr/>
  </v-app>
</template>

<style scoped>
.v-main {
  background-image: url("~@/assets/img/bgMain.png");
  background-attachment: fixed;
  background-position: center;
  background-size: cover;
}
</style>

<script>
import navigation from "./components/Navigation";
import ftr from "./components/Footer";
import home from "./components/HomeSection";
import about from "./components/AboutSection";
import demo from "./components/DemoSection.vue";
import pricing from "./components/PricingSection";
import contact from "./components/ContactSection";
import ogImage from "./assets/img/one-whatsapp.png"


export default {
  name: "App",
  metaInfo: {
    title: "OneCharge",
      meta: [
        {name: "description", content: "Landing Page for OneCharge demo"},
        {property: "og:title", content: "OneCharge"},
        {property: "og:url", content: "https://onecharge.madebysven.com"},
        {property: "og:description", content: "Landing Page for OneCharge demo"},
        {property: "og:type", content: "website"},
        {property: "og:locale", content: "nl_NL"},
        {property: "og:image", itemprop:"image", content: "https://onecharge.madebysven.com" + ogImage},
        {property: "og:image:secure_url", content: "https://onecharge.madebysven.com" + ogImage},
        {property: "og:image:type", content: "image/png"},
      ],
  },

  components: {
    navigation,
    ftr,
    home,
    about,
    demo,
    pricing,
    contact,
  },

  data: () => ({
    fab: null,
    color: "",
    flat: null,
  }),

  created() {
    const top = window.pageYOffset || 0;
    document.title = 'OneCharge';
    if (top <= 60) {
      this.color = "transparent";
      this.flat = true;
    }
  },

  watch: {
    fab(value) {
      if (value) {
        this.color = "secondary";
        this.flat = false;
      } else {
        this.color = "transparent";
        this.flat = true;
      }
    },
  },

  methods: {
    onScroll(e) {
      if (typeof window === "undefined") return;
      const top = window.pageYOffset || e.target.scrollTop || 0;
      this.fab = top > 60;
    },
    toTop() {
      this.$vuetify.goTo(0);
    },
  },
};
</script>
