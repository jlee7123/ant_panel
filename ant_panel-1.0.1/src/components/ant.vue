<template>
  <div>
    <br />
    


    <v-container fluid class="ml-10">
      <h2>Antenna Tracker : Motor</h2>
      <br />
      <!--
    <v-row>
        <v-col>
        <h3> Pan: {{ myPan }} </h3>
        </v-col>
        <v-col>
        <h3> Tilt: {{ myTilt }} </h3>
        </v-col>
        </v-row>
    -->

        <br>


      <div>
        <v-row>
          <v-col></v-col>

            <v-col>
            <Progress
              strokeColor="#FF00AA"
              :transitionDuration="2500"
              :radius="50"
              :strokeWidth="15"
              value="13"
            >
              <div class="content">{{ myPan }}</div>
              <template v-slot:footer>
                <b>PAN</b>

              </template>
            </Progress>
          </v-col>

          <v-col>
            
            <Progress
              strokeColor="#AAFF00"
              :transitionDuration="2500"
              :radius="50"
              :strokeWidth="15"
              value="60"
            >
              <div class="content">
                {{ myTilt }}
              </div>
              <template v-slot:footer>
                <b>TILT</b>
              </template>
            </Progress>
          </v-col>



          <!--
          <v-col>
            <v-progress-circular
              :rotate="360"
              :size="100"
              :width="15"
              :model-value="value"
              color="teal"
            >
              {{ value }}
            </v-progress-circular>
          </v-col>
          -->


        </v-row>
      </div>

      <br />

      <br /><br />
      <h2>Antenna Tracker : Control Panel</h2>
      <br />
      <v-card
        class="pa-10"
        outlined
        shaped
        elevation="3"
        height="450"
        width="750"
        min-height="450"
        min-width="750"
      >
        <v-row>
          <v-col>
            <v-btn
              color="orange"
              large
              block
              outlined
              raised
              rounded
              elevation="2"
              @click="doRun"
            >
              RUN
            </v-btn>

            <br />
          </v-col>
          <v-col></v-col>
          <v-col></v-col>
        </v-row>

        <v-row>
          <v-col></v-col>
          <v-col>
            <v-btn
              color="deep-purple"
              large
              block
              outlined
              raised
              rounded
              elevation="2"
              v-on="{ mousedown: tiltUp, mouseup: tiltUpStop }"
            >
              TILT UP
            </v-btn>

            <br />
          </v-col>
          <v-col></v-col>
        </v-row>

        <v-row>
          <v-col>
            <v-btn
              color="indigo"
              large
              block
              outlined
              raised
              rounded
              elevation="2"
              v-on="{ mousedown: panDown, mouseup: panDownStop }"
            >
              PAN DOWN
            </v-btn>

            <br />
          </v-col>
          <v-col>
            <v-btn
              color="pink"
              large
              block
              outlined
              raised
              rounded
              elevation="2"
              @click="init"
            >
              INIT
            </v-btn>

            <br />
          </v-col>
          <v-col>
            <v-btn
              color="indigo"
              large
              block
              outlined
              raised
              rounded
              elevation="2"
              v-on="{ mousedown: panUp, mouseup: panUpStop }"
            >
              PAN UP
            </v-btn>

            <br />
          </v-col>
        </v-row>

        <v-row>
          <v-col></v-col>
          <v-col>
            <v-btn
              color="deep-purple"
              large
              block
              outlined
              raised
              rounded
              elevation="2"
              v-on="{ mousedown: tiltDown, mouseup: tiltDownStop }"
            >
              TILT DOWN
            </v-btn>
          </v-col>
          <v-col></v-col>
        </v-row>
      </v-card>
    </v-container>
  </div>
</template>

<script>
import mqtt from "mqtt";
import { nanoid } from "nanoid";
import Progress from "easy-circular-progress";

export default {
  name: "app",
  components: {
    Progress,
  },

  data() {
    return {
      interval: {},
      //value: 10,

      //location data
      loca_obj: {
        myLatitude: 37.403625,
        myLongitude: 127.16177,
        myAltitude: 0,
      },

      myPan: 13,
      myTilt: 60,

      // mqtt 연결부
      client: {
        connected: false,
        loading: false,
      },
      connection: {
        host: "127.0.0.1",
        port: 8883,
        //port: 1883,
        endpoint: "",
        clean: true,
        connectTimeout: 4000,
        reconnectPeriod: 4000,
        clientId: "jiho_" + nanoid(15),
        username: "keti_muv",
        password: "keti_muv",
      },

      getDataTopic: {
        pan: "/Ant_Tracker/Motor_Pan",
        tilt: "/Ant_Tracker/Motor_Tilt",
      },

      motorControlTopic: "/Ant_Tracker/Control",
      //motorLocationTopic: "/Ant_Tracker/Location_Data",

      // mqtt end

      console_text_message: "",
      chk: "",

      response_text: "",
      rev_connted: false,
    };
  },
  methods: {
    sendloca: function () {
      console.log(JSON.stringify(this.loca_obj));
      this.doPublish(this.motorLocationTopic, JSON.stringify(this.loca_obj));
    },

    tiltUp: function () {
      this.doPublish(this.motorControlTopic, "tilt_up");
    },
    tiltUpStop: function () {
      this.doPublish(this.motorControlTopic, "stop");
    },
    tiltDown: function () {
      this.doPublish(this.motorControlTopic, "tilt_down");
    },
    tiltDownStop: function () {
      this.doPublish(this.motorControlTopic, "stop");
    },
    panDown: function () {
      this.doPublish(this.motorControlTopic, "pan_down");
    },
    panDownStop: function () {
      this.doPublish(this.motorControlTopic, "stop");
    },
    panUp: function () {
      this.doPublish(this.motorControlTopic, "pan_up");
    },
    panUpStop: function () {
      this.doPublish(this.motorControlTopic, "stop");
    },
    doRun: function () {
      this.doPublish(this.motorControlTopic, "run");
    },
    init: function () {
      this.doPublish(this.motorControlTopic, "init");
    },
    // onMessageHandler(topic, message) {
    //     let chkTopic = topic.substr(0, 7);
    //
    //     if (chkTopic === "/oneM2M") {
    //         console.log(message.toString());
    //     }
    // },

    createConnection() {
      console.log("createConnection () 시작!");
      if (this.client.connected) {
        this.destroyConnection();
      }

      if (!this.client.connected) {
        this.client.loading = true;
        this.connection.clientId = "mqttjs_" + "jiho" + "_" + nanoid(15);
        this.connection.host = "127.0.0.1";
        const { host, port, endpoint, ...options } = this.connection;
        const connectUrl = `ws://${host}:${port}${endpoint}`;
        try {
          this.client = mqtt.connect(connectUrl, options);

          this.client.on("connect", () => {
            console.log(connectUrl, "Connection succeeded!");
            console.log(host, "Connection succeeded!");

            this.client.connected = true;
            this.client.loading = false;

            this.doSubscribe(this.getDataTopic.pan);
            console.log("pan subscribe !!!! ");
            this.doSubscribe(this.getDataTopic.tilt);
            console.log("tilt subscribe !!!! ");

            // let subtopic = "/oneM2M/req/Mobius2/" + "Sketi" + "/#";
            //
            // this.doUnSubscribe(subtopic);
            //
            // setTimeout(() => {
            //     this.doSubscribe(subtopic);
            //     console.log("Subscribe to ", subtopic);
            // }, 200);
          });

          this.client.on("error", (error) => {
            console.log("Connection failed", error);

            this.destroyConnection();
          });

          this.client.on("close", () => {
            console.log("Connection closed");

            this.destroyConnection();

            this.connection.clientId = "mqttjs_" + "jiho" + "_" + nanoid(15);
          });

          this.client.on("message", (topic, message) => {
            console.log("Received " + message.toString() + " From " + topic);

            if (topic === this.getDataTopic.pan) {
              this.myPan = parseFloat(message.toString()).toFixed(1);
            } else if (topic === this.getDataTopic.tilt) {
              this.myTilt = parseFloat(message.toString()).toFixed(1);
            }
          });
        } catch (error) {
          console.log("mqtt.connect error", error);
          this.client.connected = false;

          setTimeout(this.createConnection, 2000);
        }
      }
    },
    doSubscribe(topic) {
      if (this.client.connected) {
        const qos = 0;
        this.client.subscribe(topic, { qos }, (error) => {
          if (error) {
            console.log("Subscribe to topics error", error);
            return;
          }

          console.log("Subscribe to topics (", topic, ")");
        });
      }
    },

    doUnSubscribe(topic) {
      if (this.client.connected) {
        this.client.unsubscribe(topic, (error) => {
          if (error) {
            console.log("Unsubscribe error", error);
          }

          console.log("Unsubscribe to topics (", topic, ")");
        });
      }
    },

    doPublish(topic, payload) {
      if (this.client.connected) {
        this.client.publish(topic, payload, 0, (error) => {
          if (error) {
            console.log("Publish error", error);
          }
        });
      }
    },

    destroyConnection() {
      if (this.client.connected) {
        try {
          if (Object.hasOwnProperty.call(this.client, "__ob__")) {
            this.client.end();
          }
          this.client = {
            connected: false,
            loading: false,
          };
          console.log(this.name, "Successfully disconnected!");
        } catch (error) {
          console.log("Disconnect failed", error.toString());
        }
      }
    },
  },
  created() {
    console.log("created () 시작!");
    this.createConnection();
  },

  mounted() {
    console.log("mounted () 시작!");

    this.interval = setInterval(() => {
      if (this.value === 100) {
        return (this.value = 0);
      }
      this.value += 10;
    }, 2000);
  },

  /*
    beforeUnmount () {
      clearInterval(this.interval)
    },
*/

  beforeDestroy() {
    this.destroyConnection();
  },
};
</script>

<style lang="scss">
#app {
  text-align: left;
  height: 100vh;
  color: #fff;
  background: #38364b;
  display: flex;
  justify-content: center;
  align-items: center;
}
body {
  margin: 0;
  padding: 0;
}


.v-progress-circular {
  margin: 1rem;
}
</style>
