<script setup>
import { ref, onMounted, watch } from "vue";
const publc_stun_list = [
  { urls: "stun:stun.l.google.com:19302" },
  { urls: "stun:stun1.l.google.com:19302" },
  { urls: "stun:stun2.l.google.com:19302" },
  { urls: "stun:stun3.l.google.com:19302" },
  { urls: "stun:stun4.l.google.com:19302" },
];

const peer = ref(null);

const serverlist = ref([publc_stun_list[0]]);
const uri = ref("");
const username = ref("");
const credential = ref("");

const candidatelist = ref([]);

function addServer() {
  if (uri.value) {
    serverlist.value.push({
      urls: uri.value,
      username: username.value,
      credential: credential.value,
    });
    uri.value = "";
    username.value = "";
    credential.value = "";
  } else {
    alert("please input uri");
  }
}
function remove(index) {
  serverlist.value.splice(index, 1);
}
function reset() {
  serverlist.value = [publc_stun_list[0]];
  uri.value = "";
  username.value = "";
  credential.value = "";
}

async function gatherCandidates() {
  candidatelist.value = [];
  if (peer.value) {
    peer.value.close();
    peer.value.onicecandidate = null;
    peer.value = null;
  }

  const startTime = Date.now();

  peer.value = new RTCPeerConnection({
    iceServers: serverlist.value,
  });

  peer.value.onicecandidate = (event) => {
    if (!event.candidate) return;
    console.log("onicecandidate", event.candidate);
    event.candidate.time = ((Date.now() - startTime) / 1000).toFixed(3);
    candidatelist.value.push(event.candidate);
  };
  peer.value.addTransceiver("audio", { direction: "sendrecv" });
  const offer = await peer.value.createOffer();
  peer.value.setLocalDescription(offer);
}

onMounted(() => {
  // restore serverlist from localStorage
  const store_serverlist = window.localStorage.getItem("serverlist");
  if (store_serverlist) {
    serverlist.value = JSON.parse(store_serverlist);
  } else {
    serverlist.value = [publc_stun_list[0]];
  }

  gatherCandidates();
});

watch(
  serverlist,
  () => {
    window.localStorage.setItem("serverlist", JSON.stringify(serverlist.value));
  },
  { immediate: false, deep: true }
);
</script>

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

    <form @submit.prevent>
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
