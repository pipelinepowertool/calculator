<template>
  <section id="download">
    <v-container fluid>
      <v-row align="center" justify="center">
        <v-col cols="10">
          <v-row align="center" justify="center">
            <v-col sm="4" class="hidden-xs-only">
              <v-img src="@/assets/img/ill2.svg" class="d-block ml-auto mr-auto" max-width="350px"/>
            </v-col>
            <v-col cols="12" sm="8" class="white--text text-left">
              <h1 class="font-weight-light display-2 mb-2">Demo</h1>
              <v-form v-if="step === 1" ref="form" v-model="valid" :lazy-validation="lazy">
                <v-text-field
                    v-model="name"
                    :rules="nameRules"
                    label="Naam/Kenteken"
                    dark
                    :disabled="rfid == null"
                    required
                ></v-text-field>
                <v-text-field
                    v-model="phone"
                    :rules="phoneRules"
                    label="Telefoonnummer (optioneel)"
                    dark
                    :disabled="rfid == null"
                    required
                ></v-text-field>

              </v-form>
              <v-btn v-if="step === 1"
                     :disabled="!valid || rfid == null"
                     rounded
                     outlined
                     large
                     color="white"
                     class="mt-4"
                     @click="submit1">
                <v-icon class="mr-2">
                  mdi-send
                </v-icon>
                Volgende
              </v-btn>

              <div v-if="step === 2">
                Pair binnen 5 minuten de tag met de OneCharge Payment Device
              </div>
              <v-progress-circular style="margin-top: 20px" v-if="step === 2 && polling === true" indeterminate color="primary">

              </v-progress-circular>
<!--              <v-btn-->
<!--                  v-if="step === 2"-->
<!--                  rounded-->
<!--                  outlined-->
<!--                  large-->
<!--                  color="white"-->
<!--                  class="mt-4"-->
<!--                  @click="submit2">-->
<!--                <v-icon class="mr-2">-->
<!--                  mdi-send-->
<!--                </v-icon>-->
<!--                Volgende-->
<!--              </v-btn>-->

              <v-form v-if="step === 3" ref="form" v-model="valid" :lazy-validation="lazy">

                <v-text-field
                    v-model="refuelled"
                    type="number"
                    :rules="refuelledRules"
                    label="Hoeveel kWh heb je bijgeladen?"
                    dark
                    required
                ></v-text-field>

              </v-form>
              <v-btn
                  v-if="step === 3"
                  :disabled="!valid"
                  rounded
                  outlined
                  large
                  color="white"
                  class="mt-4"
                  @click="submit3">
                <v-icon class="mr-2">
                  mdi-send
                </v-icon>
                Volgende
              </v-btn>

              <div v-if="step === 4">
                De betaling is geslaagd! <br>
                Zie de betaling in het dashboard en krijg de factuur van vandaag in je mailbox.
              </div>
              <v-form v-if="step === 4" ref="form" onsubmit="return false">

                <v-text-field
                    v-model="email"
                    label="E-mail"
                    dark
                    required
                ></v-text-field>

              </v-form>
              <v-btn
                  v-if="step === 4 && !!!this.email"
                  rounded
                  outlined
                  large
                  color="white"
                  class="mt-4"
                  @click="submit4">
                <v-icon class="mr-2">
                  mdi-send
                </v-icon>
                Nieuwe betaling
              </v-btn>
              <v-btn
                  v-if="step === 4 && !!this.email"
                  :disabled="ajaxInProgress"
                  rounded
                  outlined
                  large
                  color="white"
                  class="mt-4"
                  @click="submit4">
                <v-icon class="mr-2">
                  mdi-send
                </v-icon>
                Email Factuur
              </v-btn>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
    </v-container>
    <v-snackbar
        v-model="snackbar.enabled"
        timeout="3000"
        right
        top
        :color="snackbar.color"
    >
      {{ snackbar.text }}

      <template v-slot:action="{ attrs }">
        <v-btn
            text
            v-bind="attrs"
            @click="snackbar.enabled = false"
        >
          Sluiten
        </v-btn>
      </template>
    </v-snackbar>
  </section>
</template>

<script>
import axios from 'axios'

export default {
  mounted() {
    let urlParams = new URLSearchParams(window.location.search);
    const rfid = urlParams.get('rfid')
    if (rfid && this.rfidTags.includes(rfid)) {
      this.rfid = rfid
    }
  },
  data: () => ({
    ajaxInProgress: false,
    polling: true,
    valid: true,
    step: 1,
    name: "",
    nameRules: [
      (v) => !!v || "Naam is verplicht",
    ],
    rfid: null,
    rfidTags: [
      "02928d0d-523e-47f5-bb0a-aded145a2e57",
      "6cda37e4-9f62-4408-bc85-574767d55381",
      "102a96ab-1d8f-4fc7-8db8-8e8ece3daef6",
      "e611df37-acba-43b6-8211-17b0896f7a5a",
      "04969c88-e9ab-4f55-bf6c-fb1a40b47018",
      "05a7cd0a-f737-4762-a217-2ee29221042a",
      "7aa38cbf-0c9d-4395-9d87-7a72a75f41a7",
      "6d7b7a9c-341e-4567-ba10-cc6f0c54b7f3"
    ],
    refuelled: 100,
    refuelledRules: [
      (v) => !!v || "kWh is verplicht",
      (v) => (v && v >= 10) || "kWh mag minimaal 10 zijn",
      (v) => (v && v <= 500) || "kWh mag maximaal 500 zijn",
    ],
    phone: "",
    phoneRules: [
      (v) => (v === "" || /^(((\+31|0|0031)6)[1-9]\d{7})$/.test(v)) || "Incorrect nummer"
    ],
    email: "",
    lazy: false,
    snackbar: {
      enabled: false,
      text: '',
      color: ''
    },
    refuelOperation: ""
  }),
  methods: {
    submit1() {
      const data = {
        account: "2a0f562f-8ed4-4a6f-81bd-ee5cf5a52f6c",
        username: this.name,
        rfid: this.rfid,
        phone: this.phone
      }
      axios.post('https://onechargeapi.madebysven.com/users', data).then(response => {
        this.snackbar.text = response.data.message
        this.snackbar.color = "success"
        this.snackbar.enabled = true
        this.step = 2;
      }).catch(errorResponse => this.printError(errorResponse.message))
    },
    submit2() {
      axios.get('https://onechargeapi.madebysven.com/rfid/' + this.rfid).then(response => {
        this.refuelOperation = response.data
        this.snackbar.text = "Betalingsverzoek gevonden"
        this.snackbar.color = "success"
        this.snackbar.enabled = true
        this.step = 3;
      }).catch(errorResponse => {
        if (errorResponse.response.status === 409) {
          this.snackbar.text = "Maak eerst een betalingsverzoek met je tag!"
          this.snackbar.color = "error"
          this.snackbar.enabled = true
        } else {
          this.printError(errorResponse.message)
        }
      })
    },
    pollRfidEndpoint() {
      this.polling = true;
      axios.get('https://onechargeapi.madebysven.com/rfid/' + this.rfid).then(response => {
        this.refuelOperation = response.data
        this.snackbar.text = "Betalingsverzoek gevonden"
        this.snackbar.color = "success"
        this.snackbar.enabled = true
        this.step = 3;
        this.polling = false;
      }).catch(errorResponse => {
        if (errorResponse.response.status === 409) {
          setTimeout(() => this.pollRfidEndpoint(), 5000)
        } else {
          this.printError(errorResponse.message)
          this.polling = false;
          this.step = 1
        }
      })
    },
    submit3() {

      const data = {
        refuelAmount: this.refuelled,
        pricePerKwh: 0.473,
      }
      axios.put('https://onechargeapi.madebysven.com/refuel-operation/' + this.refuelOperation, data).then(response => {
        this.snackbar.text = "Bedankt voor je betaling!"
        this.snackbar.color = "success"
        this.snackbar.enabled = true
        this.step = 4;
      }).catch(errorResponse => this.printError(errorResponse.message))

    },
    submit4() {
      if (!!!this.email) {
        this.step = 1
      } else if (!/.+@.+\..+/.test(this.email)) {
        this.snackbar.text = "E-mail adres is incorrect"
        this.snackbar.color = "error"
        this.snackbar.enabled = true
      } else {
        const data = {
          information: {
            email: this.email,
            day: 26,
            month: 1,
            year: 2023,
            account: "2a0f562f-8ed4-4a6f-81bd-ee5cf5a52f6c"
          }
        }
        this.ajaxInProgress = true
        axios.post('https://onechargeapi.madebysven.com/invoice', data).then(response => {
          this.snackbar.text = "Factuur is verstuurd naar je email"
          this.snackbar.color = "success"
          this.snackbar.enabled = true
        }).catch(errorResponse => {
          this.printError(errorResponse.message)
        }).finally(() => {
          this.ajaxInProgress = false
          this.step = 1
        })

      }
    },
    printError(message) {
      this.snackbar.text = message
      this.snackbar.color = "error"
      this.snackbar.enabled = true
    }
  },
  watch: {
    step(step) {
      if (step === 2) {
        this.pollRfidEndpoint();
      }
    }
  }
};
</script>
<style scoped>
#download {
  background-image: url("~@/assets/img/bgDownload.jpg");
  background-attachment: fixed;
  background-repeat: no-repeat;
  background-size: cover;
  background-position: center;
  height: 500px;
}

#download .container,
#download .row {
  height: 100%;
}
</style>