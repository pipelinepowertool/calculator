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
              <h1 class="font-weight-light display-2 mb-2">Calculator</h1>
              <v-form v-if="step === 1" ref="form" v-model="valid" :lazy-validation="lazy">

<!--                STEP ONE-->
                <v-select
                label="Selecteer een CI/CD Tool"
                dark
                :items="['Jenkins', 'A̶z̶u̶r̶e̶D̶e̶v̶o̶p̶s̶ (coming soon)']"
                required
                v-model="selectedCicdTool">
                </v-select>
              </v-form>

              <div v-if="selectedCicdTool == 'Jenkins'">
                <v-form ref="form" :lazy-validation="lazy">
                  <v-text-field
                      v-model="jenkinsChoices.jobName"
                      type="text"
                      label="Job"
                      dark
                      required
                  ></v-text-field>
                </v-form>
                <v-form ref="form" :lazy-validation="lazy" v-if="jenkinsChoices.jobName != ''">
                  <v-text-field
                      v-model="jenkinsChoices.branchName"
                      type="text"
                      label="Git Branch"
                      dark
                      required
                  ></v-text-field>
                </v-form>
                <v-form ref="form" :lazy-validation="lazy" v-if="jenkinsChoices.jobName != ''">
                  <v-text-field
                      v-model="jenkinsChoices.buildNumber"
                      type="number"
                      label="Build number"
                      dark
                      required
                  ></v-text-field>
                </v-form>
                <v-form ref="form" :lazy-validation="lazy">
                </v-form>
                <v-form ref="form" v-model="valid" :lazy-validation="lazy">
                  <v-text-field
                      v-model="commonChoices.agentCountry"
                      type="text"
                      label="Agent host land"
                      :rules="[countryRules]"
                      dark
                      required
                  ></v-text-field>
                </v-form>
              </div>

              <v-btn v-if="step === 1"
                     :disabled="!valid || selectedCicdTool != 'Jenkins'"
                     rounded
                     outlined
                     large
                     color="white"
                     class="mt-4"
                     style="align-self: end"
                     @click="submit1">
                <v-icon class="mr-2">
                  mdi-send
                </v-icon>
                Calculate
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
import eventBus from "@/components/eventbus";

export default {
  mounted() {
    let urlParams = new URLSearchParams(window.location.search);
    const rfid = urlParams.get('rfid')
    if (rfid && this.rfidTags.includes(rfid)) {
      this.rfid = rfid
    }
  },
  data: () => ({
    self: this,
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
    selectedCicdTool: "Jenkins",
    refuelled: "",
    jenkinsChoices: {
      jobName: "",
      branchName: "",
      buildNumber: ""
    },
    commonChoices: {
      agentCountry: "NL",
    },
    countryRules: (v) => {
      try {
        const s = new Intl.DisplayNames(['en'], {type: 'region'}).of(v);
        return s.length > 2;
      } catch (e) {
        return false;
      }
    },
    email: "",
    lazy: false,
    snackbar: {
      enabled: false,
      text: '',
      color: ''
    },
  }),
  methods: {
    submit1() {
      let args = "";
      if (this.jenkinsChoices.jobName != "") {
        args += "?job=" + this.jenkinsChoices.jobName;
        if (this.jenkinsChoices.branchName != "") {
          args += "&branch=" + this.jenkinsChoices.branchName;
        }
        if (this.jenkinsChoices.buildNumber != "") {
          args += "&build=" + this.jenkinsChoices.buildNumber;
        }

      }
      eventBus.$emit('custom-event', args)

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
  height: 800px;
}

#download .container,
#download .row {
  height: 100%;
}
</style>