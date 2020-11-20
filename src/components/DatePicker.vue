<template>
  <v-row>
    <v-col
    
    >
      <v-menu
          ref="menu1"
          v-model="menu1"
          :close-on-content-click="false"
          transition="scale-transition"
          offset-y
          min-width="290px"
      >
        <template v-slot:activator="{ on, attrs }">
          <v-text-field
              v-model="dateRangeText"
              label="Picker in menu"
              prepend-icon="mdi-calendar"
              readonly
              v-bind="attrs"
              v-on="on"
          ></v-text-field>
        </template>
        <v-date-picker
            v-model="dates"
            no-title
            scrollable
            range
            @update:picker-date="menu2 = false"
            @change="onDateRangeChange"
        >
        </v-date-picker>
      </v-menu>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: "DatePicker",
  props: {
    defaultDates: {
      type: Array,
      default:()=> []
    }
  },
  data: () => ({
    menu1: false,
    menu2: false,
    dates: []
  }),
   created () {
    this.dates= this.defaultDates
    // console.log(this.dates,"dats")
  },
  // mounted () {
  //   console.log('mounted:', this.defaultDates)
  //   this.dates = this.defaultDates;
  // },
  // beforeUpdate () {
  //   console.log('beforeUpdate:', this.defaultDates)
  // },
  computed: {
    dateRangeText () {
      if (this.dates) {
        return this.dates.join(' ~ ');
      }
      return ""
    },
 

  },
  methods: {
    onDateRangeChange(){
      this.$emit('onDateRangeChange',this.dates)
    }
  }
}
</script>

<style scoped>
.v-date-picker{
      width: 400px;
}

</style>
