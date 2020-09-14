<template>
  <div>
    <h1>Educ Nat année scolaire 2019-2020</h1>
     <div>
        <table v-if="msg" style="text-align:left">
            <thead>
                <tr>
                    <th>Nom</th>
                    <th>Effectif</th>
                    <th>Adresse</th>
                    <th>Type</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="item in msg" v-bind:key="item.recordid">
                    <td>{{ item.fields.nom_etablissement }}</td>
                    <td>{{ item.fields.nombre_d_eleves }}</td>
                    <td>{{ item.fields.adresse }} <br /> {{ item.fields.code_postal }} {{ item.fields.localite_acheminement }}</td>
                    <td>{{ item.fields.secteur_d_enseignement }}</td>
                </tr>
            </tbody>
        </table>
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
      msg: null
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
    }
  }
}
</script>
