<template>
  <div class = "breakdown">
    <h1 class = "title is-3 is-spaced"> Patient Breakdown by Diagnosis </h1>
    <h2 class="subtitle"> {{breakdownWarning}} </h2>
    <canvas id = "patient-breakdown" width = "450" height = "300"> </canvas>
  </div>
</template>

<script>
import axios from 'axios';
import colorScheme from 'color-scheme';
export default {
  name: 'breakdown',
  data() {
      return {
        breakdownWarning: '',
      }
  },
  methods: {
      getInfo() {
        var self = this;
        // Return patients who are associated with the medical professional
        axios.get(this.$store.getters.patientBreakdownURL+this.$store.getters.medicalCode)
        .then(function(response) {
          // If there are no diagnoses returned
          if(response.data.length == 0) {
            self.breakdownWarning = 'Sorry, you need to add patients before you can view reports';
          } else {
          // Object that will hold the diagnosis count
          var conditionCount = {};
          // Loop through the patient diagnoses that was returned. In the conditionCount object, use the diagnoses as keys
          for (var d of response.data) {
            if(!(d in conditionCount)) {
              // If the diagnosis does not exist as a key, add it and set the initial value to 1
              conditionCount[d] = 1;
            } else {
              // Increment an value of a condition
              conditionCount[d]+=1;
            }
          }
          
         var scheme = new colorScheme;
         var palette = scheme.from_hue(21).scheme('triade').variation('default').colors();
         // Need to append a '#' at the front of each hex code generated because it is required for the colors on Chart JS and color-scheme does not do this
         for(var i = 0; i < palette.length; i++) {
           palette[i] = '#'+palette[i];
         }
         
         new Chart (document.getElementById("patient-breakdown").getContext('2d'), {
            type: 'doughnut',
            data: {
              // Use the names of the conditions as the labels
              labels: Object.keys(conditionCount),
              datasets: [{
                // Use the number of patients with that condition as the data values
                data: Object.values(conditionCount),
                backgroundColor: palette
              }]
            },
            options: {
              responsive: false,
              maintainAspectRatio: true,
              animation: {
                duration: 1000
              },
              pieceLabel: {
                render: 'percentage',
                precision: 2,
                position: 'border',
                fontSize: 14,
                fontStyle: 'bold',
                fontColor: '#fff',
              },
              legend: {
                display: true,
                position: "left",
                labels: {fontSize: 14}
              },
              tooltips: {
                callbacks: {
                  label: function(tooltipItems, data) {
                    return data.labels[tooltipItems.index]+': '+data.datasets[0].data[tooltipItems.index] + ' patient(s)'
                  }
                }
              }
            }
          })
        }
       
        })
        .catch(function (err) {
          self.breakdownWarning = 'Sorry. Information for this report cannot be displayed at this time. Try again later.';
        })
      }
  },
  mounted() {
    this.getInfo();
  }
}
</script>

<style lang="scss" scoped>
  .subtitle {
      margin-left: 2%;
  }
</style>

