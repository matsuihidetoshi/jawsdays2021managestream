<template>
  <div class="stream">
    <v-btn
      color="success"
      @click="newStreamForm = true"
    >
      new stream
    </v-btn>

    <v-dialog
      v-model="newStreamForm"
    >
      <v-card>
        <v-card-title
          class="
            headline
            grey
            lighten-2
          "
        >
          Create new stream
        </v-card-title>

        <v-card-text>
          <v-form>
            <v-text-field
              v-model="title"
              label="Title"
            />

            <v-text-field
              v-model="url"
              label="Endpoint URL"
            />

            <v-textarea
              v-model="description"
              label="Description"
            />

            <v-checkbox
              v-model="active"
              label="Active"
            />
          </v-form>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="warning"
            @click="newStreamForm = false"
          >
            cancel
          </v-btn>

          <v-btn
            color="success"
            @click="newStreamForm = false, createStream()"
          >
            create
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-row>
      <v-col
        v-for="(stream, index) in streams"
        :key="index"
        cols="12"
        md="6"
      >
        <v-card
          class="
            pa-3
            breakline
          "
        >
          <v-form>
            <v-text-field
              v-model="stream.title"
              label="Title"
            />

            <v-text-field
              v-model="stream.url"
              label="Endpoint URL"
            />

            <v-textarea
              v-model="stream.description"
              label="Description"
            />

            <v-checkbox
              v-model="stream.active"
              label="Active"
            />

            <v-row>
              <v-spacer />

              <v-btn
                color="error"
                @click="deleteConfirmation = true, deleteIndex = index"
                class="mr-2"
              >
                delete
              </v-btn>

              <v-dialog
                v-model="deleteConfirmation"
              >
                <v-card>
                  <v-card-title>
                    Confirmation: Delete
                  </v-card-title>

                  <v-card-text>
                    Are you sure?
                  </v-card-text>

                  <v-divider></v-divider>

                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                      color="warning"
                      @click="deleteConfirmation = false"
                    >
                      cancel
                    </v-btn>

                    <v-btn
                      color="error"
                      @click="deleteConfirmation = false, deleteStream(deleteIndex)"
                    >
                      delete
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>

              <v-btn
                color="primary"
                @click="updateStream(index)"
                class="mr-2"
              >
                update
              </v-btn>
            </v-row>
          </v-form>
        </v-card>
      </v-col>
    </v-row>

    <v-overlay
      :value="loading"
    >
      <v-progress-circular
        indeterminate
        :size="80"
        :width="10"
      />
    </v-overlay>

    <v-snackbar
      v-model="snackbar"
      color="blue-grey"
      top
    >
      {{ message }}
      <template v-slot:action="{ attrs }">
        <v-btn
          color="#FF9900"
          class="
            font-weight-bold
          "
          v-bind="attrs"
          @click="snackbar = false"
        >
          close
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>
<script>
import { API, graphqlOperation} from "aws-amplify"
import { createStream, updateStream, deleteStream } from "../graphql/mutations"
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
      active: false,
      stream: null,
      streams: [],
      owner: "",
      limit: 2 ** 31 - 1,
      newStreamForm: false,
      deleteConfirmation: false,
      deleteIndex: null,
      loading: false,
      snackbar: false,
      message: null/* ,
      mocks: [
        {
          url: "test1.example.com",
          title: "test session1",
          description: "this is \n test description1",
          active: false
        },
        {
          url: "test2.example.com",
          title: "test session2",
          description: "this is \n test description2",
          active: false
        },
        {
          url: "test3.example.com",
          title: "test session3",
          description: "this is \n test description3",
          active: false
        },
        {
          url: "test3.example.com",
          title: "test session3",
          description: "this is \n test description3",
          active: false
        }
      ] */
    }
  },
  mounted: function () {
    this.owner = this.$store.state.user.username
    this.displayStreams()
  },
  methods: {
    createStream: async function () {
      this.loading = true
      const stream = {
        url: this.url,
        title: this.title,
        description: this.description,
        active: this.active
      }
      try {
        this.url = this.title = this.description = ""
        this.active = false
        await API.graphql(graphqlOperation(createStream, {input: stream}))
        this.loading = false
        this.message = "Created"
        this.snackbar = true
      } catch (error) {
        error
        this.loading = false
        this.message = "Create failed"
        this.snackbar = true
      }
    },
    updateStream: async function (index) {
      this.loading = true
      const stream = {
        id: this.streams[index].id,
        url: this.streams[index].url,
        title: this.streams[index].title,
        description: this.streams[index].description,
        active: this.streams[index].active
      }
      try {
        await API.graphql(
          graphqlOperation(
            updateStream,
            {input: stream}
          )
        )
        this.loading = false
        this.message = "Updated"
        this.snackbar = true
      } catch (error) {
        error
        this.loading = false
        this.message = "Update failed"
        this.snackbar = true
      }
    },
    deleteStream: async function (index) {
      this.loading = true
      const stream = {
        id: this.streams[index].id
      }
      try {
        await API.graphql(
          graphqlOperation(
            deleteStream,
            {input: stream}
          )
        )
        this.loading = false
        this.message = "Deleted"
        this.snackbar = true
      } catch (error) {
        error
        this.loading = false
        this.message = "Delete failed"
        this.snackbar = true
      }
      this.displayStreams()
    },
    displayStreams: async function () {
      let streams = await API.graphql(graphqlOperation(
        listStreams, {limit: this.limit}
      ))
      this.streams = _.orderBy(streams.data.listStreams.items, 'createdAt', 'asc')
      
      API.graphql(
        graphqlOperation(onCreateStream, {limit: this.limit, owner: this.owner})
      ).subscribe({
        next: (eventData) => {
          const stream = eventData.value.data.onCreateStream
          const streams = [...this.streams, stream]
          this.streams = _.orderBy(streams, 'createdAt', 'asc')
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
