<template>
  <div class="main">
    <h1>Educ Nat</h1>
    <div v-if="msg">
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
      <a href="#" v-bind:class="{active: sortKey==='nombre_d_eleves'}"
              v-on:click="sortBy('nombre_d_eleves')">
              Trier par effectif croissant ou décroissant
      </a>
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
  props: {
    msg: String
  },
  data() {
    return {
      msg: null,
      reverse: false,
      searchName: '',
      type: 'All',
      zip: '',
      max: 4,
      sortKey: 'nombre_d_eleves',
      anneeScolaire: '2019-2020'
    }
  },
  beforeMount(){
    this.getName(this.anneeScolaire);
  },
  methods: {
    async getName(anneeScolaire){
      const res = await fetch('https://data.education.gouv.fr/api/records/1.0/search/?dataset=fr-en-effectifs-second-degre&q&facet=annee_scolaire&facet=academie&facet=type_d_etablissement&refine.annee_scolaire=' + anneeScolaire + '&refine.academie=PARIS&refine.type_d_etablissement=LYCEE%20D%20ENSEIGNEMENT%20GENERAL&rows=50&sort=nombre_d_eleves');
      const data = await res.json();
      this.msg = data.records;
    },
    sortBy: function(sortKey) {
      this.reverse = this.sortKey==sortKey? !this.reverse : false
      this.sortKey = sortKey
    },
    filterByName : function(record) {
      // no search, don't filter :
      if (this.searchName.length === 0) {
        return true;
      }

      return  (record.fields.nom_etablissement.toLowerCase().indexOf(this.searchName.toLowerCase()) > -1);
    },
    filterByType : function(record) {
      // no search, don't filter :
      if (this.type === "All") {
        return true;
      }

      return  record.fields.secteur_d_enseignement.toLowerCase() == this.type;
    },
    filterByZip : function(record) {
      // no zip, don't filter :
      if (this.zip === "") {
        return true;
      }

      return  record.fields.code_postal == this.zip;
    },
    orderBy : function (a, b) {
      let condition = (a[this.sortKey] > b[this.sortKey]);
      if (this.reverse) {
        return !condition;
      } else {
        return condition;
      }
    },
    queryForKeywords: function(event) {
      this.$nextTick(() => {
        if (this.annee.length > 3) {
         return this.getName((this.annee - 1) + '-' + this.annee);
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
    arrondissements () {
      const distinct_code_postaux = [...new Set(this.msg.map(record => record.fields.code_postal))];

      return distinct_code_postaux.sort(function(a, b) {
        return a - b;
      });
    }
  }
}
</script>
