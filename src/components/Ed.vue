<template>
  <div class="main">
    <h1>educ nat</h1>
    <div v-if="msg">
      <div class="filters">
        <div class="form">
          <input class="annee form" v-model="annee" placeholder="Année scolaire" :maxlength="max" @input="queryForKeywords" />
        </div>
        <div class="form">
          <input class="form" v-model="searchName" placeholder="Etablissement recherché">
        </div>
        <div class="form">
          <label><input type="radio" value="All"  v-model="type" checked>All</label>
          <label><input type="radio" value="public" v-model="type">Public</label>
          <label><input type="radio" value="privé" v-model="type">Privé</label>
        </div>
        <div class="form">
          <select v-model="zip" >
            <option value="">Tous les arrondissements</option>
            <option v-for="item in arrondissements" :key="item.key" :value="item" >{{ item }}</option>
          </select>
        </div>
        <div class="form">
          <button v-bind:class="{active: sortKey==='nombre_d_eleves'}"
                  v-on:click="sortBy('nombre_d_eleves')">
                  Trier par effectif
          </button>
          <button v-bind:class="{active: sortKey==='code_postal'}"
                  v-on:click="sortBy('code_postal')">
                  Trier par code postal
          </button>
        </div>
      </div>
    <table class="table">
      <tbody>
        <tr v-for="record in filteredRecords" :key="record.recordid">
            <td class="case">{{ record.fields.nom_etablissement }}</td>
            <td class="case">{{ record.fields.nombre_d_eleves }}</td>
            <td class="case">{{ record.fields.adresse }}</td>
            <td class="case">{{ record.fields.code_postal }}</td>
            <td class="case">{{ record.fields.secteur_d_enseignement }}</td>
        </tr>
      </tbody>
    </table>
    </div>

    <div v-else>
      <p>L'éductaion nationale ne répond plus</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Ed',
  data() {
    return {
      msg: null,
      reverse: false,
      searchName: '',
      type: 'All',
      annee: '',
      zip: '',
      max: 4,
      sortKey: 'code_postal',
      anneeScolaire: '2019-2020'
    }
  },
  beforeMount() {
    this.getName();
  },
  methods: {
    async getName () {
      const res = await fetch('https://data.education.gouv.fr/api/records/1.0/search/?dataset=fr-en-effectifs-second-degre&q&facet=annee_scolaire&facet=academie&facet=type_d_etablissement&refine.annee_scolaire=' + this.anneeScolaire + '&refine.academie=PARIS&refine.type_d_etablissement=LYCEE%20D%20ENSEIGNEMENT%20GENERAL&rows=50&sort=' + this.sortKey + '');
      const data = await res.json();
      this.msg = data.records;
    },
    sortBy: function (sortKey) {
      this.reverse = this.sortKey==sortKey? !this.reverse : false
      this.sortKey = sortKey
      this.getName()
    },
    filterByName: function (record) {
      if (this.searchName.length === 0) {
        return true;
      }

      return  (record.fields.nom_etablissement.toLowerCase().indexOf(this.searchName.toLowerCase()) > -1);
    },
    filterByType: function (record) {
      if (this.type === 'All') {
        return true;
      }

      return  record.fields.secteur_d_enseignement.toLowerCase() === this.type;
    },
    filterByZip: function (record) {
      if (this.zip === '') {
        return true;
      }
      return  record.fields.code_postal == this.zip;
    },
    orderBy: function (a, b) {
      let condition = (a[this.sortKey] > b[this.sortKey]);
      if (this.reverse) {
        return !condition;
      } else {
        return condition;
      }
    },
    queryForKeywords: function (event) {
      this.$nextTick(() => {
        if (this.annee.length === 0) {
          this.anneeScolaire = '2019-2020';
          return this.getName();
        }
        else if (this.annee.length > 3) {
          this.anneeScolaire = (this.annee - 1) + '-' + this.annee;
          return this.getName();
        }

      });
    }
  },
  computed: {
    filteredRecords: function () {
      return this.msg
      .filter(this.filterByName)
      .filter(this.filterByType)
      .filter(this.filterByZip)
      .sort(this.orderBy);
    },
    arrondissements() {
      const distinct_code_postaux = [...new Set(this.msg.map(record => record.fields.code_postal))];
      return distinct_code_postaux.sort(function(a, b) {
        return a - b;
      });
    }
  }
}
</script>
