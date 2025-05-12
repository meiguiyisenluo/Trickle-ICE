<template>
  <div>
    <h1>Trickle-ICE</h1>
    <h2>STUN/TURN server serverlist</h2>
    <div class="serverlist">
      <ul>
        <li v-for="(item, index) in serverlist" :key="index">
          {{ item.urls }}
          <template v-if="item.username && item.credential"
            >[{{ item.username }}:{{ item.credential }}]</template
          >
          <button @click="remove(index)">remove</button>
        </li>
      </ul>
    </div>

    <form>
      <div style="margin-bottom: 10px">
        <label for="stun">STUN or TURN URI:</label>
        <input type="text" v-model="uri" />
      </div>
      <div style="margin-bottom: 10px">
        <label for="stun">TURN username:</label>
        <input type="text" v-model="username" />
      </div>
      <div style="margin-bottom: 10px">
        <label for="stun">TURN credential:</label>
        <input type="text" v-model="credential" />
      </div>
      <button @click="addServer">add server</button>
      <button @click="reset">reset to default</button>
      <button @click="gatherCandidates">gather candidates</button>
    </form>
    <table
      border="1"
      cellspacing="0"
      cellpadding="0"
      style="width: 100%; border-collapse: collapse"
    >
      <thead>
        <tr>
          <th>time</th>
          <th>type</th>
          <th>foundation</th>
          <th>protocol</th>
          <th>address</th>
          <th>port</th>
          <th>priority</th>
          <th>url(if present)</th>
          <th>relayProtocol(if present)</th>
          <th>candidate</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in candidatelist" :key="index">
          <td>{{ item.time }}</td>
          <td>{{ item.type }}</td>
          <td>{{ item.foundation }}</td>
          <td>{{ item.protocol }}</td>
          <td>{{ item.address }}</td>
          <td>{{ item.port }}</td>
          <td>{{ item.priority }}</td>
          <td>{{ item.url }}</td>
          <td>{{ item.relayProtocol }}</td>
          <td>{{ item.candidate }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script>
const publc_stun_list = [
  { urls: "stun:stun.l.google.com:19302" },
  { urls: "stun:stun1.l.google.com:19302" },
  { urls: "stun:stun2.l.google.com:19302" },
  { urls: "stun:stun3.l.google.com:19302" },
  { urls: "stun:stun4.l.google.com:19302" },
];
export default {
  data() {
    return {
      peer: null,

      serverlist: [publc_stun_list[0]],
      uri: "",
      username: "",
      credential: "",

      candidatelist: [],
    };
  },
  watch: {
    // store serverlist to localStorage
    serverlist: {
      handler() {
        window.localStorage.setItem(
          "serverlist",
          JSON.stringify(this.serverlist)
        );
      },
      deep: true,
    },
  },
  async created() {
    // restore serverlist from localStorage
    const serverlist = window.localStorage.getItem("serverlist");
    if (serverlist) {
      this.serverlist = JSON.parse(serverlist);
    } else {
      this.serverlist = [publc_stun_list[0]];
    }

    this.gatherCandidates();
  },

  methods: {
    addServer() {
      if (this.uri) {
        this.serverlist.push({
          urls: this.uri,
          username: this.username,
          credential: this.credential,
        });
        this.uri = "";
        this.username = "";
        this.credential = "";
      } else {
        alert("please input uri");
      }
    },
    remove(index) {
      this.serverlist.splice(index, 1);
    },
    reset() {
      this.serverlist = [publc_stun_list[0]];
      this.uri = "";
      this.username = "";
      this.credential = "";
    },

    async gatherCandidates() {
      this.candidatelist = [];
      if (this.peer) {
        this.peer.close();
        this.peer.onicecandidate = null;
        this.peer = null;
      }

      const startTime = Date.now();

      this.peer = new RTCPeerConnection({
        iceServers: this.serverlist,
      });

      this.peer.onicecandidate = (event) => {
        if (!event.candidate) return;
        console.log("onicecandidate", event.candidate);
        event.candidate.time = ((Date.now() - startTime) / 1000).toFixed(3);
        this.candidatelist.push(event.candidate);
      };
      this.peer.addTransceiver("audio", { direction: "sendrecv" });
      const offer = await this.peer.createOffer();
      this.peer.setLocalDescription(offer);
    },
  },
};
</script>

<style lang="scss">
.serverlist {
  margin-bottom: 20px;
  box-shadow: inset 0 0 5px #ccc;
  max-height: 100px;
  overflow-y: auto;
  li {
    padding: 5px;
  }
}

form {
  margin-bottom: 20px;
}

th {
  padding: 5px;
  text-align: center;
  background-color: #f0f0f0;
  border: 1px solid #ccc;
}
td {
  font-size: 0.8rem;
  padding: 5px;
  text-align: center;
  border: 1px solid #ccc;
}
</style>
