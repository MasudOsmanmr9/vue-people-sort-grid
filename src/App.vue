<template>
<div v-if="rowData !== null" style="height: 100%">
  <b-container class="bv-example-row pt-5">
    <b-row class="pb-3">
      <b-col cols="12" class="d-flex justify-content-between">
          <h4>Start Training System</h4>
          <div class="d-flex align-items-center timer-sec">
            <h4 class="mr-3 mb-0" v-if="timerDisplay != null">{{timerDisplay}}</h4>
            <h4 class="mr-3 mb-0" v-else>Successfully sorted!!</h4>
            <b-button class="ml-1" @click="showModal" variant="outline-primary">Start sorting</b-button>
          </div>
      </b-col>
    </b-row>

    <b-row class="">
      <b-col cols="12">
        <ag-grid-vue style="width: 100%; height: 500px;"
          class="ag-theme-alpine"
          :columnDefs="columnDefs"
          :rowDragManaged="true"
          :animateRows="true"
          :rowData="slicedRowData"
          @grid-ready="onGridReady"
          @rowDragEnd="modifyRowData"
          >
        </ag-grid-vue>
      </b-col>
    </b-row>
  </b-container>
  <b-modal ref="my-modal" hide-footer title="How Many People?">
      <div class="d-block text-center">
        <h3>Enter a number of how many people you want to add to the list</h3>
        <b-form-input :state="peopleNumberCheck" v-model="numberOfPeople" aria-describedby="input-live-help input-live-feedback" placeholder="Enter a number"></b-form-input>
        <b-form-invalid-feedback id="input-live-feedback">
          Enter number between 5-{{rowData.length-5}}
        </b-form-invalid-feedback>
        <b-form-text id="input-live-help">click start to start the timer.</b-form-text>
      </div>
      <div class="w-100 d-flex justify-content-end">
        <b-button class="mt-3" variant="warning" block @click="startTimer" >Start</b-button>
      </div>
  </b-modal>  
</div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import { AgGridVue } from "ag-grid-vue";
import dataSource from './json/sorceData.js';
export default {
  name: 'App',
  components: {
    AgGridVue,
  },
  beforeMount() {
    this.gridOptions = {};
    this.columnDefs = [
            { field: 'email', rowDrag: true, checkboxSelection:true },
            { field: 'potatoes'},
            { field: 'full_name'},
            { field: 'last_name'},
            { field: 'athlete' },
            { field: 'country' },
            { field: 'year', width: 100 },
            { field: 'date' },
            { field: 'sport' },
            { field: 'gold' },
            { field: 'silver' },
            { field: 'bronze' },
          ];
    this.rowData = dataSource;
    this.numberOfPeople = this.rowData.length;
  },
  data: function () {
    return {
      gridOptions: null,
      gridApi: null,
      columnApi: null,
      columnDefs: null,
      rowData: null,
      timerInterval : null,
      timerDisplay : '00:00',
      numberOfPeople: null,
    };
  },
  created() {
    //console.log('created sourec data ', sourceData);
  },
  computed: {
    peopleNumberCheck() {
      return this.numberOfPeople >= 5 && this.numberOfPeople <= this.rowData.length-5 ? true : false
    },
    slicedRowData(){
      return this.rowData.slice(0, this.numberOfPeople);
    }
  },
  methods: {
    modifyRowData(data){
        var itemsToUpdate = [];
        let temp = null;
        let _this = this
        this.gridApi.forEachNodeAfterFilterAndSort(function (rowNode) {
          itemsToUpdate.push(rowNode.data);
        });
        this.rowData = itemsToUpdate;
        if(this.isSorted(this.rowData)){
          this.timerDisplay = null;
          clearInterval(this.timerInterval);
        }
    },
    isSorted(data){
      for(let i=1; i < data.length;i++){
        // console.log('compairing',data[i].year,data[i-1].year)
        if(data[i].potatoes>data[i-1].potatoes){
          return false;
        }
      }
      return true;
    },
    onGridReady(params) {
      this.gridApi = params.api;
      this.gridColumnApi = params.columnApi;
     // console.log(this.gridApi)
    },
    startTimer(){
      this.hideModal();
      //console.log('accessing')
      var  minutes = 0, seconds = 1;
      this.timerInterval = setInterval(()=>{
        // console.log(this.timerDisplay)
          if(seconds == 60){
            minutes += 1;
            seconds = 0;
          }
          let min = minutes < 10 ? "0" + minutes : minutes;
          let sec = seconds < 10 ? "0" + seconds : seconds;
          this.timerDisplay = min + ":" + sec;

          seconds++;
      }, 1000);
    },
    showModal() {
      this.$refs['my-modal'].show()
    },
    hideModal() {
      this.$refs['my-modal'].hide()
    },
  },
}
</script>

<style lang="scss">
  @import "~ag-grid-community/dist/styles/ag-grid.css";
  @import "~ag-grid-community/dist/styles/ag-theme-alpine.css";

  .timer-sec{
    h4{
      margin-right: 10px;
    }
  }

</style>
