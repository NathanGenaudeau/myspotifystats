<script setup lang="ts">
import { ref } from 'vue';

interface Song {
  ts: string,
  username: string,
  platform: string,
  ms_played: number,
  conn_country: string,
  ip_addr_decrypted: string,
  user_agent_decrypted: string,
  master_metadata_track_name: string,
  master_metadata_album_artist_name: string,
  master_metadata_album_album_name: string,
  spotify_track_uri: string,
  episode_name: string,
  episode_show_name: string,
  reason_start: string,
  reason_end: string,
  shuffle: boolean,
  skipped: boolean,
  offline: boolean,
  offline_timestamp: string,
  incognito_mode: string,
}

const data = ref<any>();
const dataParsed = ref<Song[]>([]);
const list = ref<any>([]);

const readAsText = async (file: any) => {
		return new Promise((resolve, _)=>{
			const fileReader = new FileReader();
			fileReader.onload = function(){
				return resolve({data: fileReader.result, name:file.name, size: file.size, type: file.type});
			}
			fileReader.readAsText(file);
		})
}

const upload = async (input: any) => {
  const files : Blob[] = [...input.target.files];
  data.value = await Promise.all(files.map(f => { return readAsText(f) }));
  dataParsed.value = data.value.flatMap((r : any) => JSON.parse(r.data)).filter((song: Song) => song.ms_played >= 20000);

  // regroupement par musique pour voir la plus écoutée
  list.value = Object.values(dataParsed.value.reduce(
    (r: any, c: Song) => {
      r[c.spotify_track_uri] = r[c.spotify_track_uri] || {id: c.spotify_track_uri, name: c.master_metadata_track_name, artist: c.master_metadata_album_artist_name, total_time: Math.trunc(c.ms_played/1000), count: 1};
      r[c.spotify_track_uri].total_time += Math.trunc(c.ms_played/1000);
      r[c.spotify_track_uri].count += 1;
      return r;
    },{}));

  list.value.sort((a: any, b: any) => b.count - a.count);
}
</script>

<template>
  <div>
    <input @change="upload" type="file" multiple/>
  </div>

  <div v-if="list.length > 0" v-for="(song, index) in list">
    <div>n°{{index+1}} : {{ song.name }} de {{ song.artist }} - {{ song.count }} écoutes soit {{ (song.total_time/3600).toFixed(2) }}h</div>
  </div>
</template>

<style scoped>
</style>
