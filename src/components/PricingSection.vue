<template>
  <section id="pricing" class="pb-8" v-show="canLoad">
    <v-container fluid>
      <v-row align="center" justify="center">
        <v-col cols="10">
          <v-card style="width: 100%">
            <h1 class="text-center pt-6 font-weight-light display-2">Verbruik</h1>
            <v-divider class="my-6"></v-divider>
            <v-row class="text-center" justify="center">
              <v-col
                  cols="2"
                  class="py-2">
                <v-text-field
                    v-model="pricePerKwh"
                    type="number"
                    label="Prijs per kWh in €"
                    required
                ></v-text-field>
              </v-col>
            </v-row>

            <v-row class="text-center">
              <v-col
                  cols="12"
                  class="py-2">
                <v-btn-toggle
                    mandatory
                    v-model="selectedGroup"
                    :rounded=true
                    color="deep-purple-accent-3"
                    group>
                  <v-btn value="0">
                    Totaal
                  </v-btn>
                  <v-btn value="1" v-if="responseBody.co2PerHour != null">
                    Per Run
                  </v-btn>
                  <v-btn value="2" v-if="responseBody.co2PerHour != null">
                    Per Uur
                  </v-btn>
                </v-btn-toggle>
              </v-col>
            </v-row>

            <v-row class="text-center" justify="center">
              <v-col class="col-12 col-sm-6 col-md-4">
                <div class="flex-center">
                  <v-card-text v-for="data in simpleData" :key="data.label" v-if="!data.hideInSelectedGroup()">
                    <div class="text-uppercase text-h4  mt-6 blue--text">{{ data.calculation() }}</div>
                    <div class="text-uppercase blue--text">{{data.label}}</div>
                  </v-card-text>
                </div>
              </v-col>
            </v-row>

            <v-row class="text-center" justify="center" v-if="pricePerKwh != 0">
              <v-col class="col-12 col-sm-6 col-md-4">
                <div class="flex-center">
                  <v-card-text>
                    <div class="flex-center">
                      <div class="circle1">
                        <div class="circle2">
                          <v-img src="~@/assets/img/profit.png"></v-img>
                        </div>
                      </div>
                    </div>
                    <div class="text-uppercase text-h4  mt-6 blue--text">€ {{ calculatePrice }}</div>
                    <div class="text-uppercase blue--text">Uitgegeven</div>

                  </v-card-text>
                </div>
              </v-col>
            </v-row>
            <v-row class="text-center">
              <v-col class="col-12 col-sm-6 col-md-4" v-for="equivalent in equivalencies" :key="equivalent.img">
                <div class="flex-center">
                  <v-card-text>
                    <div class="flex-center">
                      <div class="circle1">
                        <div class="circle2">
                          <v-img :src="getImgUrl(equivalent.img)"></v-img>
                        </div>
                      </div>
                    </div>
                    <div class="text-uppercase text-h4  mt-6 blue--text">{{ equivalent.calculation([0]) }}</div>
                    <div class="text-uppercase blue--text">
                      {{ equivalent.label }}
                      <v-btn icon style="justify-content: left" @click="goToUrl(equivalent.source)">
                        <v-icon color="info" size="large">mdi-information-outline</v-icon>
                      </v-btn>
                    </div>
                  </v-card-text>
                </div>
              </v-col>
            </v-row>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
    <div class="svg-border-rounded text-light">
      <svg
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 1440 320"
          preserveAspectRatio="none"
          fill="currentColor"
      >
        <path
            d="M0,64L80,90.7C160,117,320,171,480,181.3C640,192,800,160,960,138.7C1120,117,1280,107,1360,101.3L1440,96L1440,320L1360,320C1280,320,1120,320,960,320C800,320,640,320,480,320C320,320,160,320,80,320L0,320Z"
        ></path>
      </svg>
    </div>
  </section>
</template>
<style lang="scss">
$main_color: #283e79;

ul {
  font-size: 13px;
  line-height: 1.5em;
  margin: 5px 0 15px;
  padding: 0;

  li {
    list-style: none;
    position: relative;
    padding: 0 0 0 20px;
  }

  li {
    &::before {
      content: "";
      position: absolute;
      left: 0;
      top: 5px;
      width: 10px;
      height: 10px;
      background-color: $main_color;
      border-radius: 50%;
      -moz-border-radius: 50%;
      -webkit-eeborder-radius: 50%;
    }
  }
}
</style>

<style scoped>
.header {
  background-color: #283e79;
  color: white;
}

.circle1 {
  border-radius: 50%;
  width: 150px;
  height: 150px;
  background-color: #f0f8ff;
  display: flex;
  align-items: center;
  justify-content: center;
}

.circle2 {
  border-radius: 50%;
  width: 100px;
  height: 100px;
  background-color: #e0effc;
  display: flex;
  align-items: center;
  justify-content: center;
}

.flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

.svg-border-rounded svg {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  color: #f4f7f5;
  z-index: -1;
}

#pricing {
  z-index: 0;
}

.content {
  z-index: 1;
}

svg {
  overflow: hidden;
}

section {
  position: relative;
}
</style>

<script>
import eventBus from "@/components/eventbus";
import axios from "axios";

export default {
  mounted() {
    eventBus.$on('custom-event', (args) => {
      axios.get("https://api.madebysven.com/calculator/jenkins" + args).then(response => {
        this.responseBody = response.data
        this.canLoad = true;
        eventBus.$emit('load-calculations-navigation')
      })

    })
  },
  beforeDestroy() {
    // removing eventBus listener
    eventBus.$off('custom-event')
  },
  data() {
    return {
      refresh: 0,
      canLoad: false,
      pricePerKwh: 0,
      groups: {
        co2: ["co2Total", "co2PerRun", "co2PerHour"],
        kwh: ["kwhTotal", "kwhPerRun", "kwhPerHour"],
      },
      selectedGroup: 0,
      responseBody: {
        kwhTotal: null,
        kwhPerRun: null,
        kwhPerHour: null,
        pipelineRuns: null,
        runtime: null,
        co2Total: null,
        co2PerRun: null,
        co2PerHour: null,
        cpuUtilization: null,
      },

    }
  },
  computed: {
    calculatePrice() {
      const kwh = this.responseBody[this.groups.kwh[this.selectedGroup]];
      return this.prettyPrintNumber(kwh * this.pricePerKwh)
    },
    simpleData() {
      return [
        {
          label: "CO2",
          calculation: () => {
            const co2 = this.responseBody[this.groups.co2[this.selectedGroup]];
            if (co2 > 1000) {
              return parseFloat(co2/1000).toFixed(2) + "kg";
            }
            return co2 + "g"
          },
          hideInSelectedGroup: () => false
        },
        {
          label: "kWh",
          calculation: () => this.responseBody[this.groups.kwh[this.selectedGroup]],
          hideInSelectedGroup: () => false
        },
        {
          label: "Pipeline Runs",
          calculation: () => {
            if (this.selectedGroup == 0) {
              return this.responseBody.pipelineRuns;
            }
            if (this.selectedGroup == 2) {
              return this.prettyPrintNumber(this.calculateRunsPerHour(this.responseBody.pipelineRuns, this.responseBody.runtime))
            }
            if (this.selectedGroup == 1) {
              return 1;
            }
          },
          hideInSelectedGroup: () => this.selectedGroup == 1 ? true : false
        },
        {
          label: "CPU-gebruik",
          calculation: () => this.responseBody.cpuUtilization + "%",
          hideInSelectedGroup: () => false
        },
        {
          label: "Verbruiksuren",
          calculation: () => {
            const runtimePerHour = this.responseBody.runtime / 3600;
            if (this.selectedGroup == 0) {
              return runtimePerHour.toFixed(0)
            }
            if (this.selectedGroup == 2) {
              return runtimePerHour > 1 ? 1 : runtimePerHour;
            }
            if (this.selectedGroup == 1) {
              return this.prettyPrintNumber(this.calculateRunTimePerRun(this.responseBody.pipelineRuns, this.responseBody.runtime))
            }
          },
          hideInSelectedGroup: () => this.selectedGroup == 2 ? true : false
        }
      ]
    },
    equivalencies() {
      this.refresh;
      return [
        {
          label: "Kilometer gereden met een elektrische auto",
          img: "electric-car.png",
          source: "https://tesla360.nl/tesla-model-3-hoeveel-kwh-per-100-km/",
          calculation: () => this.prettyPrintNumber(this.calculateKmTravelElectricCar(this.responseBody[this.groups.kwh[this.selectedGroup]]))
        },
        {
          label: "Kilometer gereden met een benzine auto",
          img: "gas-station.png",
          source: "https://vandebron.nl/blog/gemiddeld-stroomverbruik",
          calculation: () => this.prettyPrintNumber(this.calculateKmTravelGasolineCar(this.responseBody[this.groups.co2[this.selectedGroup]]))
        },
        {
          label: "Kilometer gevlogen met een vliegtuig",
          img: "plane.png",
          source: "https://www.milieucentraal.nl/duurzaam-vervoer/co2-uitstoot-fiets-ov-en-auto",
          calculation: () => this.prettyPrintNumber(this.calculateKmTravelAirplane(this.responseBody[this.groups.co2[this.selectedGroup]]))
        },
        {
          label: "Elektriciteit verbruikt van een standaard huishouden per dag",
          img: "house.png",
          source: "https://theicct.org/publication/co2-emissions-from-commercial-aviation-2013-2018-and-2019",
          calculation: () => this.prettyPrintNumber(this.calculateEnergyOfHomesUsed(this.responseBody[this.groups.kwh[this.selectedGroup]]))
        },
        {
          label: "Smartphones opgeladen",
          img: "smartphone.png",
          source: "https://www.epa.gov/energy/greenhouse-gases-equivalencies-calculator-calculations-and-references#smartphones",
          calculation: () => this.prettyPrintNumber(this.calculateSmartphonesCharged(this.responseBody[this.groups.kwh[this.selectedGroup]]))
        },
        {
          label: "Kilometer afgelegd met een vrachtschip (van een 2kg pakketje)",
          img: "freight.png",
          source: "https://greenandgrumpy.com/whats-the-carbon-footprint-of-a-cup-of-tea",
          calculation: () => this.prettyPrintNumber(this.calculateKmTravel2KgPackageFreight(this.responseBody[this.groups.co2[this.selectedGroup]]))
        },

        {
          label: "Kilo aardappelen geproduceerd",
          img: "potato.png",
          source: "https://www.co2everything.com/co2e-of/potatoes",
          calculation: () => this.prettyPrintNumber(this.calculatePotatoesProduced(this.responseBody[this.groups.co2[this.selectedGroup]]))
        },
        {
          label: "Glazen wijn geproduceerd",
          img: "wine-glass.png",
          source: "https://www.co2everything.com/co2e-of/freight-shipping",
          calculation: () => this.prettyPrintNumber(this.calculateGlassesOfWineProduced(this.responseBody[this.groups.co2[this.selectedGroup]]))
        },
        {
          label: "Kopjes thee gemaakt",
          img: "tea.png",
          source: "https://www.co2everything.com/co2e-of/wine",
          calculation: () => this.prettyPrintNumber(this.calculateBoilingCupsOfWater(this.responseBody[this.groups.co2[this.selectedGroup]]))
        },
        // {
        //   label: "Kilometers gereden met een elektrische auto",
        //   calculation: (v) => self.calculateGrBeefProduced(co2gr)
        // },
      ]
    },

    size() {
      const size = {md: "large", xl: "x-large"}[
          this.$vuetify.breakpoint.name
          ];
      return size ? {[size]: true} : {};
    },
    sizeTwo() {
      return this.$vuetify.breakpoint.smAndUp;
    },

  },
  methods: {
    refreshCalc() {
      this.refresh++;
    },
    goToUrl(url) {
      window.open(url);
    },
    getImgUrl(pic) {
      return require('../assets/img/usage/' + pic)
    },
    prettyPrintNumber(nr) {
      return parseFloat(nr.toFixed(2)).toLocaleString("nl-NL")
    },
    calculateKmTravelElectricCar(kwh) {
      return kwh / 17.5; //https://tesla360.nl/tesla-model-3-hoeveel-kwh-per-100-km/
    },
    calculateEnergyOfHomesUsed(kwh) {
      return kwh / 7.56; //https://vandebron.nl/blog/gemiddeld-stroomverbruik
    },
    calculateKmTravelGasolineCar(co2gr) {
      return co2gr / 149; //https://www.milieucentraal.nl/duurzaam-vervoer/co2-uitstoot-fiets-ov-en-auto
    },
    calculateKmTravelAirplane(co2gr) {
      return co2gr / 101; //https://theicct.org/publication/co2-emissions-from-commercial-aviation-2013-2018-and-2019
    },
    calculateSmartphonesCharged(kwh) {
      return kwh / 0.012; //https://www.epa.gov/energy/greenhouse-gases-equivalencies-calculator-calculations-and-references#smartphones
    },
    calculateBoilingCupsOfWater(co2gr) {
      return co2gr / 23; //https://greenandgrumpy.com/whats-the-carbon-footprint-of-a-cup-of-tea
    },

    calculatePotatoesProduced(co2gr) {
      return co2gr / 50; //https://www.co2everything.com/co2e-of/potatoes
    },
    calculateKmTravel2KgPackageFreight(co2gr) {
      return co2gr / 0.03; //https://www.co2everything.com/co2e-of/freight-shipping
    },
    calculateGlassesOfWineProduced(co2gr) {
      return co2gr / 130; //https://www.co2everything.com/co2e-of/wine
    },
    calculateGrBeefProduced(co2gr) {
      return co2gr / 155; //https://www.co2everything.com/co2e-of/beef
    },
    calculateRunsPerHour(runs, runtime) {
      return runs / (runtime / 3600);
    },
    calculateRunTimePerRun(runs, runtime) {
      return (runtime / 3600) / runs;
    }
  }
};
</script>

