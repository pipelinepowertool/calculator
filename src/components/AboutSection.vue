<template>
  <section id="about">
    <v-container fluid>
      <v-row align="center" justify="center">
        <v-col cols="10">
          <v-row align="center" justify="center">
            <v-col cols="12" md="7">
              <h1 class="font-weight-light display-1 mb-3">
                Wij geloven in één Europees betaalsysteem voor laadsystemen, zodat alle klanten ontzorgd worden
              </h1>
              <v-row>
                <v-col cols="12" class="d-flex align-center">
                  <v-img
                    src="@/assets/img/green-energy.png"
                    max-width="60px"
                    class="mr-4"
                  />
                  <p class="text-justify">
                    OneCharge = duurzaamheid.
                    Met ons systeem heb je maar één pasje of tag nodig om in heel Europa op te laden.
                    Hierdoor wordt het elektrisch rijden gestimuleerd en al onze pasjes komen uit gerecycled materiaal.
                  </p>
                </v-col>
                <v-col cols="12" class="d-flex align-center">
                  <v-img
                    src="@/assets/img/icon1.svg"
                    max-width="60px"
                    class="mr-4"
                  />
                  <p class="text-justify">
                    Met OneCharge bespaart u geld door het afsluiten van één abonnement in plaats van allerlei verschillende abonnementen.
                    Dit abonnement is in heel Europa geldig.
                  </p>
                </v-col>
                <v-col cols="12" class="d-flex align-center">
                  <v-img
                    src="@/assets/img/monitor.png"
                    max-width="60px"
                    class="mr-4"
                  />
                  <p class="text-justify">
                    Door middel van onze dashboards komt u niet voor verrassingen te staan.
                    U ziet precies welk voertuig waar is opgeladen en de kosten.
                  </p>
                </v-col>
              </v-row>
            </v-col>
            <v-col cols="12" md="5" class="d-none d-md-flex">
              <v-img
                src="@/assets/img/ill1.svg"
                class="d-block ml-auto mr-auto"
                max-width="400px"
              />
            </v-col>
          </v-row>
        </v-col>
      </v-row>
    </v-container>
  </section>
</template>

<style scoped>
#about {
  background-color: #f4f7f5;
}
</style>
