<template>
  <div class="stream">
    <v-row>
      <v-col
        v-for="(mock, index) in mocks"
        :key="index"
        cols="12"
        md="6"
      >
        <v-card
          class="breakline"
        >
          {{ mock.title }}<br>
          {{ mock.url }}<br>
          {{ mock.description }}
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>
<script>
import { API, graphqlOperation} from "aws-amplify"
import { createStream } from "../graphql/mutations"
import { listStreams } from "../graphql/queries"
import { getStream } from "../graphql/queries"
import { onCreateStream } from "../graphql/subscriptions"
import _ from 'lodash'
export default {
  name: 'Stream',
  data () {
    return {
      url: "",
      title: "",
      description: "",
      stream: null,
      streams: [],
      owner: "",
      limit: 2 ** 31 - 1,
      mocks: [
        {
          url: "test1.example.com",
          title: "test session1",
          description: "this is \n test description1"
        },
        {
          url: "test2.example.com",
          title: "test session2",
          description: "this is \n test description2"
        },
        {
          url: "test3.example.com",
          title: "test session3",
          description: "this is \n test description3"
        },
        {
          url: "test3.example.com",
          title: "test session3",
          description: "this is \n test description3"
        }
      ]
    }
  },
  mounted: function () {
    this.owner = this.$store.state.user.username
    this.displayStreams()
  },
  methods: {
    createStream: async function () {
      const stream = {url: this.url, title: this.title, description: this.description}
      try {
        this.url = this.title = this.description = ""
        await API.graphql(graphqlOperation(createStream, {input: stream}))
      } catch (error) {
        error
      }
    },
    displayStreams: async function () {
      let streams = await API.graphql(graphqlOperation(
        listStreams, {limit: this.limit}
      ))
      this.streams = _.orderBy(streams.data.listStreams.items, 'updatedAt', 'desc').slice(0, 100)
      
      API.graphql(
        graphqlOperation(onCreateStream, {limit: this.limit, owner: this.owner})
      ).subscribe({
        next: (eventData) => {
          const stream = eventData.value.data.onCreateStream
          const streams = [...this.streams, stream]
          this.streams = _.orderBy(streams, 'id', 'asc').slice(0, 100)
        }
      })
    },
    singleStream: async function (selectedStream) {
      let stream = await API.graphql(graphqlOperation(
        getStream, {id: selectedStream.id}
      ))
      this.stream = stream
    }
  }
}
</script>
<style scoped>
.breakline {
  white-space: pre-wrap;
}
</style>
