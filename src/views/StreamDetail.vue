<template>
  <div>
    <v-row>
      <v-col
        cols="12"
      >
        <v-card>
          <v-card-subtitle
            v-if="stream"
          >
            URL: {{ stream.url }}
          </v-card-subtitle>

          <v-card-subtitle>
            ARN: {{ arn }}
          </v-card-subtitle>

          <v-card-subtitle>
            Channel ID: {{ channelId }}
          </v-card-subtitle>

          <v-card-subtitle
            v-if="stream"
          >
            Slug: {{ stream.slug }}
          </v-card-subtitle>

          <v-card-actions>
            <v-spacer />
            <v-btn
              v-if="stream && (!stream.state || stream.state === 0)"
              color="primary"
              @click="pushMetadataId(1)"
            >
              start question
            </v-btn>

            <v-btn
              v-else-if="stream && stream.state === 1"
              color="primary"
              @click="pushMetadataId(2)"
            >
              display result
            </v-btn>

            <v-btn
              v-else-if="stream && stream.state === 2"
              color="primary"
              @click="pushMetadataId(0)"
            >
              finish question
            </v-btn>
          </v-card-actions>

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
import axios from "axios"

export default {
  data () {
    return {
      loading: false,
      snackbar: false,
      stream: null,
      message: 'Updated',
      arn: '',
      channelId: ''
    }
  },
  mounted () {
    this.loading = true
    this.singleStream().finally(() => {
      console.log(this.stream)
      this.arn = this.convertUrlToArn(this.stream.url)
      this.setChannelId(this.stream.url)
      this.setRegion(this.stream.url)
      this.loading = false
    })
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
    },
    convertUrlToArn: function (url) {
      const region = url.split('/')[6].split('.')[0]
      const number = url.split('.')[5]
      const id = url.split('.')[7]
      return 'arn:aws:ivs:' + region + ':' + number + ':channel/' + id
    },
    setChannelId: function (url) {
      this.channelId = url.split('.')[7]
    },
    setRegion: function (url) {
      this.region = url.split('/')[6].split('.')[0]
    },
    pushMetadataId: async function (state) {
      this.stream.state = state
      this.loading = true
      let metadataId = null
      if (state === 1) {
        metadataId = new Date().getTime()
        this.stream.metadataId = metadataId
      } else {
        metadataId = this.stream.metadataId
      }
      const stream = {
        id: this.stream.id,
        metadataId: metadataId,
        state: state
      }
      try {
        await API.graphql(
          graphqlOperation(
            updateStream,
            {input: stream}
          )
        )
        this.operateQuestion(metadataId, state)
      } catch (error) {
        this.loading = false
        this.message = error
        this.snackbar = true
      }
    },
    operateQuestion: async function (metadataId, state) {
      const states = ['F', 'Q', 'R']
      try {
        const client = axios.create({
          baseURL: 'https://6aqv2bajue.execute-api.ap-northeast-1.amazonaws.com'
        })
        client.defaults.headers.common['Content-Type'] = 'application/json'
        await client.post('/default/jawsdays2021putIvsTimedMetadata', {
          channelArn: this.arn,
          metadata: states[state] + ':' + metadataId,
          region: this.region
        })
        this.loading = false
        this.message = 'Started question'
        this.snackbar = true
      } catch (error) {
        this.loading = false
        this.message = error
        this.snackbar = true
      }
    }
  }
}
</script>
