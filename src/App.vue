<template>
  <section>

  <div id="app" >
    <div v-if="page === 'ajout'">
      <section>
      <b-collapse
          aria-id="contentIdForA11y2"
          class="panel"
          animation="slide">

        <h1 class="title has-text-centered">Ajout des participants</h1>
        <div class="panel-block">
          <b-field label="Participant" position="is-centered">
            <b-input v-model="participantName"></b-input>
          </b-field>
          <div class="buttons">
            <b-button type="is-primary" expanded v-on:click="addParticipant()">Ajouter Participant</b-button>
          </div>
        </div>
        <div class="panel-block">
          <b-table>
                  <div v-for="participant of participants"   :key="participant.id" >
                    <b-datepicker-table-row>
                      <span>
                        <div class="panel-block">{{ participant.name }}
                          <b-button  type="is-danger" v-on:click="addBlackList(participant.id)">Liste noire</b-button>
                        </div>
                      </span>
                    </b-datepicker-table-row>
                  </div>
          </b-table>

        </div>
        <div class="panel-block">
          <b-button position="is-centered" type="is-success" v-on:click="goToTirage()">Passer au tirage</b-button>
        </div>
      </b-collapse>
      </section>
    </div>
    <div v-if="page === 'tirage'">
      <h1 class="title has-text-centered">Tirage au sort</h1>
      <div class="buttons">
        <b-button type="is-primary" expanded v-on:click="goToHistory()">Voir l'historique</b-button>
      </div>

      <b-collapse
          aria-id="contentIdForA11y2"
          class="panel"
          animation="slide">

        <div class="panel-block">
          <div v-if="tirages.length>0">
          {{ tirages[tirages.length-1].Participant_1}}<br>
          {{ tirages[tirages.length-1].Participant_2}}
          </div>
        </div>
      </b-collapse>



      <div class="buttons">
        <b-button type="is-success"  v-on:click="Tirer()">Tirer</b-button>
      </div>
    </div>


    <div v-if="page === 'history'">
      <h1 class="title has-text-centered">Historique du tirage au sort</h1>

      <div class="buttons">
        <b-button type="is-primary" expanded v-on:click="goToTirage()">Retourner au tirage</b-button>
      </div>
      <b-collapse
          aria-id="contentIdForA11y2"
          class="panel"
          animation="slide">


        <div class="panel-block">

          <div v-for="index in 5"   :key="index">
            <div class="panel-block" v-if="tirages.length-index>=0">
              {{ tirages[tirages.length-index].Participant_1}}<br>
              {{ tirages[tirages.length-index].Participant_2}}<br><br>
            </div>
          </div>

        </div>
      </b-collapse>


    </div>
  </div>
  </section>
</template>

<script>
import axios from "axios";



export default {
  name: "App",

  data(){
    return {
      participants: [],
      participantName: '',
      blackList: [],
      tirages: [],
      page: 'ajout'

    };
  },
  async created(){
    try{
      const res = await axios.get(`http://localhost:3000/Participants`);
      this.participants = res.data;
    }catch(e){
      console.error(e);
    }
  },
  methods:{
    async addParticipant(){
      //this.participantArr.push({name: participant, listNoir: this.blackList})

      const res = await axios.post(`http://localhost:3000/Participants`, { name: this.participantName,
        listNoir: this.blackList });
      this.participants = [...this.participants, res.data];
      this.participantName = "";
      this.blackList = [];
    },
    async addBlackList(participantID) {
      this.blackList.push(participantID);
    },
    async goToTirage(){
      this.page = 'tirage'
    },
    async goToHistory(){
      this.page = 'history'
    },
    async Tirer(){
      const nbParticipants = await axios.get(`http://localhost:3000/Participants`).then(participants =>{
        return participants.data.length;
      });
      var participant_1 = await axios.get(`http://localhost:3000/Participants`,).then(participants =>{
        return participants.data[0];
      });


      var participant_2 = null;
      do{
        var ordreParticipant_2 = parseInt(Math.random()*nbParticipants, 10);
        participant_2 = await axios.get(`http://localhost:3000/Participants/`).then(participant => {
          return participant.data[ordreParticipant_2];
        }).catch(error => {
          console.log(error)
        });
      }while(participant_2.id === participant_1.id || participant_1.listNoir.includes(participant_2.id) || participant_2 == null);

      this.addTirage(participant_1,participant_2);
      this.deleteParticipants(participant_1.id,participant_2.id);
    },
    async addTirage(p1, p2){
      const res =await axios.post(`http://localhost:3000/Tirages`, { Participant_1: p1.name,
        IdParticipant_1: p1.id,
        Participant_2: p2.name,
        IdParticipant_2: p2.id });
        this.tirages = [...this.tirages, res.data];
    },
    async deleteParticipants(id1, id2){
      await axios.delete(`http://localhost:3000/Participants/`+id1);
      await axios.delete(`http://localhost:3000/Participants/`+id2);
    },

  }
};
</script>

<style>

</style>
