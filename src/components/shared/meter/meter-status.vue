<template>

  <div class="modal meter-status-modal">

    <div class="modal-background"></div>
    <div class="modal-content meter-status-modal--form round-corners">
      <h1 class="meter-status-modal__title">Current Meter</h1>
      <div class="field">
        <label>Question: {{meter.question}}</label>
      </div>
      <div class="field">
        <label>Scale: {{meter.scale[0]}} - {{meter.scale[1]}}</label>
      </div>
      <div class="field">
        <label>Date Assigned: {{meter.due_date}}</label>
      </div>
      <button id="meter-edit" class="button button-default is-fullwidth" @click="edit">Edit Meter</button>
      <button id="meter-delete" class="button button-default meter-status-modal--create is-fullwidth" @click="deleteMeter">Delete Meter</button>
    </div>

    <button class='modal-close is-large' aria-label='close' @click='close'></button>

  </div>

</template>

<script>
import axios from "axios";
import moment from "moment";

export default {

  props: ["calendar", "meter"],

  methods: {
    edit: function() {
      document.getElementsByClassName("meter-status-modal")[0].classList.remove("is-active");
      document.getElementsByClassName("meter-edit-modal")[0].classList.add("is-active");
    },
    deleteMeter: function() {
      for(var i=0; i < this.calendar.length; i++) {
        if(this.calendar[i].date === this.meter.due_date) {
          // logical delete of meter from calendar
          this.calendar[i].meter = {};
        }
      }
      // close modal
      document.getElementsByClassName("meter-status-modal")[0].classList.remove("is-active");
      // post delete to database
      this.postDelete();
    },
    postDelete: function() {
      // define this for in post request
      let self = this;

      // get current patient username for post
      let user = this.$store.getters.getCurrentPatient.userName;
      // define meter to ensure proper format
      const meter = {
          label: this.meter.label,
          question: this.meter.question,
          scale: this.meter.scale,
          due_date: this.meter.due_date,
      }

      axios.post(this.$store.getters.deleteTreatment+user, {'treatment' : meter, user}).then(
      function(response)
      {
        if(response.status === 200){
          // remove meter from vuex
          self.$store.dispatch("deleteMeter", self.meter);
        } else {
          alert("Failed to Create Meter");
        }
      }).catch(function(err){
        throw err;
      });
    },
    close: function() {
      document.getElementsByClassName("meter-status-modal")[0].classList.remove("is-active");
    }
  }
}

</script>

<style lang="scss">

.meter-status-modal {
  &__title {
    font-size: 2em;
    text-align: center;
    font-weight: bold;
    color: $green;
  }

  &--form {
    background: $white;
    padding: 2rem;
    text-align: left;
  }

  &--create {
    margin: 1rem 0;
  }
}

</style>
