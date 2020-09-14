<template>
  <div class="main">
    <h1>Educ Nat année scolaire 2019-2020</h1>
    <div v-if="msg">
    <input v-model="searchName" class="form-control" placeholder="Etablissement recherché">
    <br/>
    <label><input type="radio" value="all"  v-model="type" checked>All</label>
    <label><input type="radio" value="public" v-model="type">Public</label>
    <label><input type="radio" value="privé" v-model="type">Privé</label>

    <br />

    <table class="table">
      <thead>
        <tr>
          <th v-for="column in columns" :key="column">
            <a  href="#" v-bind:class="{active: sortKey==column}"
              v-on:click="sortBy(column)">
              {{ column }}
            </a>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="record in filteredRecords" :key="record.recordid">
            <td class="name">{{ record.fields.nom_etablissement }}</td>
            <td class="number">{{ record.fields.nombre_d_eleves }}</td>
            <td class="address">{{ record.fields.adresse }}</td>
            <td class="zip">{{ record.fields.code_postal }}</td>
            <td class="type">{{ record.fields.secteur_d_enseignement }}</td>
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
      sortKey: 'nom_d_etablissement',
      reverse: false,
      searchName: '',
      type: 'All',
      columns: ['nom_etablissement', 'nombre_d_eleves', 'adresse', 'code_postal', 'secteur_d_enseignement'],
    }
  },
  beforeMount(){
    this.getName();
  },
  methods: {
    async getName(){
      const res = await fetch('https://data.education.gouv.fr/api/records/1.0/search/?dataset=fr-en-effectifs-second-degre&q&facet=annee_scolaire&facet=academie&facet=type_d_etablissement&refine.annee_scolaire=2019-2020&refine.academie=PARIS&refine.type_d_etablissement=LYCEE%20D%20ENSEIGNEMENT%20GENERAL&rows=50&sort=nombre_d_eleves');
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
    orderBy : function (userA, userB) {
      let condition = (userA[this.sortKey] > userB[this.sortKey]);
      if (this.reverse) {
        return !condition;
      } else {
        return condition;
      }
    }
  },
  computed: {
    filteredRecords: function () {
      return this.msg
      .filter(this.filterByName)
      .filter(this.filterByType)
      .sort(this.orderBy);
    }
  }
}
</script>
