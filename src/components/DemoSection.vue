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
      let args = "?country=" + this.commonChoices.agentCountry;
      if (this.jenkinsChoices.jobName != "") {
        args += "&job=" + this.jenkinsChoices.jobName;
        if (this.jenkinsChoices.branchName != "") {
          args += "&branch=" + this.jenkinsChoices.branchName;
        }
        if (this.jenkinsChoices.buildNumber != "") {
          args += "&build=" + this.jenkinsChoices.buildNumber;
        }
      }
      eventBus.$emit('custom-event', args)

    },
    printError(message) {
      this.snackbar.text = message
      this.snackbar.color = "error"
      this.snackbar.enabled = true
    }
  },
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