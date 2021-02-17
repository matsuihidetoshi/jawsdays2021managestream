<template>
  <div>
    <v-row>
      <v-col
        cols="12"
      >
        <v-card>
          <v-card-text>
            <v-form
              v-if="stream"
            >
              <v-text-field
                v-model="stream.title"
                label="Title"
              />

              <v-textarea
                v-model="stream.description"
                label="Description"
              />
            </v-form>
          </v-card-text>

          <v-card-actions>
            <v-spacer />

            <v-btn
              to="/"
            >
              back
            </v-btn>

            <v-btn
              color="success"
              @click="updateStream()"
            >
              update
            </v-btn>
          </v-card-actions>
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
import { updateStream } from "../graphql/mutations"
import { getStream } from "../graphql/queries"

export default {
  data () {
    return {
      loading: false,
      snackbar: false,
      stream: null,
      message: 'Updated'
    }
  },
  mounted () {
    this.loading = true
    this.singleStream().finally(() => { this.loading = false })
  },
  methods: {
    updateStream: async function () {
      this.loading = true
      const stream = {
        id: this.stream.id,
        url: this.stream.url,
        slug: this.stream.slug,
        title: this.stream.title,
        description: this.stream.description,
        active: this.stream.active
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
    singleStream: async function () {
      const response = await API.graphql(graphqlOperation(
        getStream, {id: this.$route.params.id }
      ))
      this.stream = response.data.getStream
    }
  }
}
</script>
