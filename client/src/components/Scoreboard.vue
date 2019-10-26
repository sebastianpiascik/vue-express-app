
<template>
  <div class="section section--scoreboard">
    <div v-if="!isLoading">
      <div class="section__heading">
        <p>
          <span>Klasa: {{currentHorseClass.kat}}</span>
        </p>
        <div v-on:click="getPrevClass" class="arrow arrow-prev">
          &lt;
          <span>Poprzednia</span>
        </div>
        <div v-on:click="getNextClass" class="arrow arrow-next">
          <span>Następna</span>&gt;
        </div>
      </div>
      <div class="section__table">
        <table>
          <tr>
            <td>Id</td>
            <td>Numer</td>
            <td>Nazwa konia</td>
            <td>Suma punktów</td>
            <td>Suma typ</td>
            <td>Suma ruch</td>
            <td></td>
            <td v-if="isAuth">Edycja</td>
          </tr>
          <tr
            v-for="horse in classHorses"
            v-bind:key="horse.id"
            :class="'horseRow horseRow'+horse.id"
          >
                  <td>{{ classHorses.indexOf(horse)+1 }}</td>
                  <td>{{ horse.numer }}</td>
                  <td>{{ horse.nazwa }}</td>
                  <td>{{ horse.suma.sumaPunktow }}</td>
                  <td>{{ horse.suma.typ }}</td>
                  <td>{{ horse.suma.ruch }}</td>
                  <td>
                    <span class="show-scores" v-on:click="showScores(horse)">Pokaż wyniki</span>
                  </td>
                  <td v-if="isAuth && currentHorseClass.isOpen">
                    <router-link v-bind:to="{ name: 'Judgement', params: { id: horse.id } }">oceń</router-link>
                  </td>
                  <td v-if="horse.wynik.rozjemca && isAuth">
                    <input
                      style="width:30px"
                      type="number"
                      id="rozjemca"
                      name="rozjemca"
                      v-model="horse.wynik.rozjemca"
                      min="1"
                      v-bind:max="horsesWithArbitrator.length"
                      v-if="isArbitrator"
                    >
                    <input
                      style="width:30px"
                      type="number"
                      id="rozjemca"
                      name="rozjemca"
                      v-model="horse.wynik.rozjemca"
                      min="1"
                      v-bind:max="horsesWithArbitrator.length"
                      disabled
                      v-else
                    >
                  </td>
            <td colspan="100%">
              <table class="hide">
                <tr>
                  <td>typ</td>
                  <td>glowa</td>
                  <td>kloda</td>
                  <td>nogi</td>
                  <td>ruch</td>
                </tr>
                <tr v-for="(result, index) in horse.wynik.noty" v-bind:key="index">
                  <td>{{result.typ}}</td>
                  <td>{{result.glowa}}</td>
                  <td>{{result.kloda}}</td>
                  <td>{{result.nogi}}</td>
                  <td>{{result.ruch}}</td>
                </tr>
              </table>
            </td>
          </tr>
        </table>
        <div v-if="isArbitrator">
          <button v-on:click="savePlaces">Zapisz miejsca</button>
        </div>
        <div v-if="currentHorseClass.isOpen">
          <button v-on:click="closeClass">Zakończ klase</button>
        </div>
        <div v-else>
          <p>Klasa zakonczona</p>
        </div>
      </div>
      <div class="section-results" v-if="showSingleScores">
        <p>Wyniki</p>
        <div class="section-results__element section-results__element--heading">
          <div>typ</div>
          <div>glowa</div>
          <div>kloda</div>
          <div>nogi</div>
          <div>ruch</div>
        </div>
        <div
          class="section-results__element"
          v-for="(result, index) in choosenHorse.wynik.noty"
          v-bind:key="index"
        >
          <div>{{result.typ}}</div>
          <div>{{result.glowa}}</div>
          <div>{{result.kloda}}</div>
          <div>{{result.nogi}}</div>
          <div>{{result.ruch}}</div>
        </div>
      </div>
    </div>
    <div v-else>
      <Spinner/>
    </div>
  </div>
</template>

<script>
import Spinner from "@/components/Spinner";

export default {
  components: {
    Spinner
  },
  data() {
    return {
      isLoading: true,
      currentHorseClass: null,
      isArbitrator: false,
      showSingleScores: false,
      choosenHorse: null,
      classHorses: null,
      isOpen: true,
    };
  },
  computed: {
    horseClasses() {
      return this.$store.getters.HORSE_CLASSES;
    },
    horses() {
      return this.$store.getters.HORSES;
    },
    horsesWithArbitrator() {
      return this.$store.getters.HORSES.filter(horse => horse.wynik.rozjemca);
    },
    isAuth() {
      return this.$store.getters.IS_AUTH;
    }
  },
  mounted() {
    this.getHorseClasses();
  },
  methods: {
    showScores(horse) {
      let el = document.querySelector('.horseRow'+horse.id+' table');
      // document.querySelectorAll(".horseRow").forEach(elem => {
      //   elem.
      // })
      if (el.classList.contains("hide")){
        el.classList.remove("hide");
        document.querySelector('.show-scores').innerHTML = "Ukryj";
      } else{
        el.classList.add("hide");
        document.querySelector('.show-scores').innerHTML = "Pokaż wyniki";
      }
    },
    async getHorseClasses() {
      this.currentHorseClass = this.horseClasses[0];
      this.setNewClassHorses();
    },
    closeClass(){
      this.currentHorseClass.isOpen = false;
      this.$store.dispatch("EDIT_HORSE_CLASS", this.currentHorseClass);
    },
    isNeededArbitrator() {
      let isNeeded = false;
      let wynikiRozjemca = [];
      let uniqueWyniki = [];

      this.horsesWithArbitrator.forEach(horse => {
        if (horse.wynik.rozjemca) {
          wynikiRozjemca.push(horse.wynik.rozjemca);
          isNeeded = true;
        }
      });

      uniqueWyniki = [...new Set(wynikiRozjemca)];
      if (uniqueWyniki.length === wynikiRozjemca.length) {
        isNeeded = false;
      }
      return isNeeded;
    },
    savePlaces() {
      this.horsesWithArbitrator.forEach(horse => {
        this.$store.dispatch("EDIT_HORSE", horse);
        console.log("b");
      });
      this.isLoading = true;
      this.getHorseClasses();
    },
    async setNewClassHorses() {
      this.classHorses = this.horses.filter(
        horse => parseInt(horse.klasa) === parseInt(this.currentHorseClass.id)
      );
      await this.sortClassHorses();
      this.isArbitrator = await this.isNeededArbitrator();
      this.isLoading = false;
    },
    async getNextClass() {
      this.isLoading = true;
      let index = await this.horseClasses.indexOf(this.currentHorseClass);
      if (index >= 0 && index < this.horseClasses.length - 1)
        this.currentHorseClass = this.horseClasses[index + 1];
      else this.currentHorseClass = this.horseClasses[0];

      this.setNewClassHorses();
    },
    async getPrevClass() {
      this.isLoading = true;
      let index = await this.horseClasses.indexOf(this.currentHorseClass);
      if (index > 0) this.currentHorseClass = this.horseClasses[index - 1];
      else
        this.currentHorseClass = this.horseClasses[
          this.horseClasses.length - 1
        ];

      this.setNewClassHorses();
    },
    async sortClassHorses() {
      await this.calculateHorsesPoints();
      this.classHorses.sort(this.sortByPoints);
    },
    sortByPoints(a, b) {
      if (a.suma.sumaPunktow === b.suma.sumaPunktow) {
        if (a.suma.typ === b.suma.typ) {
          if (a.suma.ruch === b.suma.ruch) {
            if (a.wynik.rozjemca && b.wynik.rozjemca) {
              return b.wynik.rozjemca - a.wynik.rozjemca;
            } else {
              a.wynik.rozjemca = 1;
              b.wynik.rozjemca = 1;
            }
          }
          return b.suma.ruch - a.suma.ruch;
        }
        return b.suma.typ - a.suma.typ;
      }
      return b.suma.sumaPunktow - a.suma.sumaPunktow;
    },
    calculateHorsesPoints() {
      this.classHorses.map(horse => {
        horse.suma = {
          sumaPunktow: 0,
          typ: 0,
          ruch: 0
        };
        horse.wynik.noty.forEach(singleResult => {
          let singleResultEntries = Object.entries(singleResult);
          for (const singleResultEntry of singleResultEntries) {
            if (singleResultEntry[0] === "typ")
              horse.suma.typ += parseFloat(singleResultEntry[1]);
            if (singleResultEntry[0] === "ruch")
              horse.suma.ruch += parseFloat(singleResultEntry[1]);
            horse.suma.sumaPunktow += parseFloat(singleResultEntry[1]);
          }
        });
      });
    }
  },
  sockets: {
    async updateScoreboard(data) {
      this.isLoading = true;
      await this.$store.dispatch("SOCKET_EDIT_HORSE", data);
      this.setNewClassHorses();
    }
  }
};
</script>