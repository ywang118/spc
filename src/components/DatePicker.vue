<template>
  <v-row>
    <v-col
        cols="12"
        sm="6"
        md="4"
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
    <!-- dates-test : {{resultDate}} -->
  </v-row>
</template>

<script>
export default {
  name: "DatePicker",
  props: {
    dates: {
      type: Array,
      default: () => {},
    }
  },
  data: () => ({
    menu1: false,
    menu2: false,
  }),
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

</style>
