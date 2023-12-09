<template>
  <form class="form flex items-center justify-center flex-wrap w-full" @submit="submitForm">
    <div class="formElement m-4 flex flex-col w-full sm:w-auto sm:flex-row sm:items-center">
      <label for="location">Location</label>
      <select class="px-10 py-4 bg-slate-600 m-4 rounded-md w-90%" id="location">
        <option
          v-for="location in locations"
          :key="location.id + location.name"
          :value="location.id"
        >
          {{ location.name }}
        </option>
      </select>
    </div>
    <div class="formElement m-4 flex flex-col w-full sm:w-auto sm:flex-row sm:items-center">
      <label for="date">When</label>
      <input
        id="date"
        type="date"
        :valueAsDate="new Date()"
        class="px-6 py-4 bg-slate-600 m-4 rounded-md w-90%"
        :min="minDate()"
      />
      <input type="time" class="px-6 py-4 bg-slate-600 m-4 rounded-md w-90%" />
    </div>
    <div class="formElement m-4 flex flex-col w-full sm:w-auto sm:flex-row sm:items-center">
      <button
        type="submit"
        class="px-6 py-4 w-90% rounded-md hover:bg-slate-600 hover:border-white border-slate-600 border-2"
      >
        Schedule a weather check
      </button>
    </div>
  </form>
</template>

<script>
import moment from 'moment'

export default {
  props: {
    locations: {
      type: Array,
      required: true
    }
  },
  setup(props, ctx) {
    function submitForm(e) {
      e.preventDefault()
      ctx.emit('submitted', {
        location: e.target.elements[0].value,
        time: e.target.elements[1].value + ' ' + e.target.elements[2].value
      })
    }

    function minDate() {
      return moment().format('YYYY-MM-DD')
    }

    return { submitForm, minDate }
  }
}
</script>
