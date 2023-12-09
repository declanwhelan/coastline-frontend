<template>
  <div v-if="!serverDown" class="flex flex-col items-center">
    <h3 class="text-4xl font-xlarge p-8">Weather Jobs</h3>

    <WeatherForm :locations="locations" @submitted="sendScheduleJob" />

    <div
      v-if="errorMessage !== ''"
      class="border border-2 border-red-700 capitalize m-4 px-6 py-4 flex items-center"
    >
      {{ errorMessage }}
    </div>

    <DataTable class="w-full" caption="Queued Jobs" :headers="['Location', 'Time', '']">
      <tr v-if="jobs.length === 0" class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
        <td class="px-6 py-4">No Jobs Scheduled</td>
        <td></td>
        <td></td>
      </tr>
      <tr
        v-else
        v-for="job in jobs"
        :key="job.id"
        class="bg-white border-b dark:bg-gray-800 dark:border-gray-700"
      >
        <td class="px-6 py-4">
          {{ locations.find((location) => location.id == job.location)?.name ?? '' }}
        </td>
        <td class="px-6 py-4">{{ getFormattedDate(job.runtime) }}</td>
        <td class="px-6 py-4 hover:text-red-700 cursor-pointer" @click="deleteJob(job.id)">
          Delete
        </td>
      </tr>
    </DataTable>

    <DataTable class="w-full" caption="Results" :headers="['Time', 'Location', 'Conditions']">
      <tr
        v-for="result in results"
        :key="result.id"
        class="bg-white border-b dark:bg-gray-800 dark:border-gray-700"
      >
        <td class="px-6 py-4">{{ getFormattedDate(result.datetime) }}</td>
        <td class="px-6 py-4">{{ result.location }}</td>
        <td class="px-6 py-4">
          <p>{{ weatherCodes[result.data.values.weatherCode] }}</p>
          <p>Temp: {{ result.data.values.temperature.toFixed() }}°C</p>
          <p>Humidity:{{ result.data.values.humidity }}%</p>
          <p>Precipitation: {{ result.data.values.precipitationProbability }}%</p>
        </td>
      </tr>
    </DataTable>
  </div>
  <div v-else>
    <h3 class="text-3xl font-xlarge p-8">
      Sorry, the server part of this application is not available. please check it is running and
      available on port 8080.
    </h3>
  </div>
</template>

<script>
import { ref, onBeforeMount } from 'vue'
import moment from 'moment'
import DataTable from './components/DataTable.vue'
import WeatherForm from './components/WeatherForm.vue'
import weatherCodes from './assets/weatherCodes.json'

const DATETIME_FORMAT = 'YYYY-MM-DD HH:mm'
const SERVER_DATETIME_FORMAT = 'YYYY-MM-DDTHH:mm'

export default {
  components: { DataTable, WeatherForm },
  setup() {
    const socket = ref(null)
    const jobs = ref([])
    const results = ref([])
    const errorMessage = ref('')
    const locations = ref([])
    const serverDown = ref(false)

    function getFormattedDate(dateStr) {
      return moment(dateStr).format(DATETIME_FORMAT)
    }

    function dateHasPassed(date) {
      return moment(date, DATETIME_FORMAT).isBefore(moment())
    }

    function sendScheduleJob(details) {
      if (moment(details.time, DATETIME_FORMAT, true).isValid()) {
        if (!dateHasPassed(details.time)) {
          errorMessage.value = ''
          socket.value.send(
            JSON.stringify({
              type: 'schedule',
              data: {
                runtime: moment(details.time).format(SERVER_DATETIME_FORMAT),
                location: details.location
              }
            })
          )
        } else {
          errorMessage.value = 'date cannot be in the past'
        }
      } else {
        errorMessage.value = 'date format is incorrect, please try again'
      }
    }

    function deleteJob(id) {
      socket.value.send(
        JSON.stringify({
          type: 'cancel',
          data: { id }
        })
      )
    }

    onBeforeMount(() => {
      socket.value = new WebSocket('ws://localhost:8080')
      socket.value.onerror = () => {
        serverDown.value = 'true'
      }
      socket.value.onopen = () => {
        socket.value.send(JSON.stringify({ type: 'status' }))
        socket.value.send(JSON.stringify({ type: 'locations' }))
      }
      socket.value.onmessage = (event) => {
        let dataObj = JSON.parse(event.data)

        switch (dataObj.type) {
          case 'status':
            jobs.value = dataObj.data.jobs
            results.value = dataObj.data.results
            break
          case 'error':
            errorMessage.value = dataObj.data
            break
          case 'locations':
            locations.value = dataObj.data
            break
          default:
            errorMessage.value = 'do not understand message from backend'
        }
      }
    })
    return {
      jobs,
      results,
      locations,
      errorMessage,
      weatherCodes,
      serverDown,
      getFormattedDate,
      sendScheduleJob,
      deleteJob
    }
  }
}
</script>
